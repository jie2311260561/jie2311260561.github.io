---
layout: post
title:  "海思个人源码"
date:   2021-8-1
categories: C Linux  hispark
tags: Using C/C++
excerpt: 个人源码记录
--- 



#include <iostream>
#include <opencv2/core.hpp>
#include <opencv2/highgui.hpp>
#include <opencv2/imgproc.hpp>
#include<opencv2/xfeatures2d.hpp>
#include<math.h>
#include <fstream>

#include "hi_ext_util.h"
#include "mpp_help.h"
#include "ai_plug.h"

#include "sample_comm_nnie.h"
#include "nnie_sample_plug.h"
// key init head file
#include <poll.h>
#include <stdio.h>
#include <stdint.h>
#include <fcntl.h>
#include <string.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/fcntl.h>

#define MSG(args...) printf(args)

using namespace std;
using namespace cv;
using namespace cv::xfeatures2d;

#define PLUG_UUID          "\"hi.meter_detect\""
#define PLUG_DESC          "\"仪表识别\""  // UTF8 encode

#define FRM_WIDTH          640
#define FRM_HEIGHT         480
#define TENNIS_OBJ_MAX     256
#define DRAW_RETC_THICK    2

static OsdSet* g_osdsTennis = NULL;
static int g_osd0Tennis = -1;
static int g_supportAudio = 0;  //创建了线程的标志位

static pthread_t g_thrdId = 0; //线程ID

//与线程相关的 结构体
static SkPair g_stmChn = {
    .in = -1,
    .out = -1
};

typedef struct tagIPC_IMAGE{
    HI_U64 u64PhyAddr;
    HI_U64 u64VirAddr;
    HI_U32 u32Width;
    HI_U32 u32Height;
} IPC_IMAGE;

//按键相关初始化 的变量
int gpio1_fd = -1;
int gpio2_fd = -1;
int ret1 	 = -1;
int ret2     = -1;
char buff[10] = {0};
struct pollfd fds1[1];
struct pollfd fds2[1];
//模式选择
static char Key_Function_mod = 0;

static const char METER_DETECT[] = "{"
    "\"uuid\": " PLUG_UUID ","
    "\"desc\": " PLUG_DESC ","
    "\"frmWidth\": " HI_TO_STR(FRM_WIDTH) ","
    "\"frmHeight\": " HI_TO_STR(FRM_HEIGHT) ","
    "\"butt\": 0"
"}";

HI_S32 frame2Mat(IPC_IMAGE *srcFrame, Mat &dstMat);
static int ThefileFeatureofimgs(Mat img);
static void* GetAudioFileName(void* arg)
{
    // Mat 
    HI_CHAR filename[30] = {} ;
	//RecogNumInfo resBuf = {0};
	int ret;
	
	while(1) {
		ret = FdReadMsg(g_stmChn.in, &filename, 30*sizeof(HI_CHAR));
        //LOGD("output srcFrm is %d\n",strlen(srcFrm));
        //LOGI("%d\n",ret);
        //LOGE("sizeof is %d\n",sizeof(VIDEO_FRAME_INFO_S));
		if (ret == (30*sizeof(HI_CHAR))) { 

            // LOGI("other srcFrm  Wight is %d\n",srcFrm.u32Width);
            // LOGI("other srcFrm  Height is %d\n",srcFrm.u32Height);
            // LOGI("other srcFrm  PhyAddr is %d\n",srcFrm.u64PhyAddr);
            // LOGI("other srcFrm  VirAddr is %d\n",srcFrm.u64VirAddr);
            LOGI("receive message from pip\n");
            LOGI("#####################################################\n");
			LOGI("filename is %s\n",filename);
            //PlayAudio(resBuf);
            Mat image = imread(filename);
            if(image.size == 0){
                LOGE("frame to Mat error \n");
            }
            else{
                LOGI("file Feature start \n");
                ThefileFeatureofimgs(image);
            } 
            cout<<image.size()<<endl;
            }
	}
}
static const char* MetersDetectProf(void)
{
    return METER_DETECT;
}
/**
    读取fd中当前的所有数据，并抛弃.
*/
static int FdReadAll(int fd)
{
    uint8_t buf[SMALL_BUF_SIZE];
    int ret;

    while ((ret = read(fd, buf, sizeof(buf))) > 0) {}
    return ret < 0 ? -errno : ret;
}

/**
    监听到button2有输入.
*/
void OnBtn2(void* user, int fd, uint32_t evts)
{
    EvtChkRet(evts, FDE_PRI | FDE_ERR, fd);
    int ret = lseek(fd, 0, SEEK_SET);
    HI_EXP_LOGE(ret < 0, "lseek BTN2 file FAIL, err='%s, %d'\n",  strerror(errno), errno);
    FdReadAll(fd);
    // LOGI("2222222222222222222222222222222222211111111111111\n");
     LOGI("2222222222222222222222222222\n");
    if(Key_Function_mod != 2){
        Key_Function_mod = 2;
    }
    else{
        Key_Function_mod = 0 ;
    }
}

/**
    监听到button3有输入.
*/
void OnBtn3(void* user, int fd, uint32_t evts)
{
    EvtChkRet(evts, FDE_PRI | FDE_ERR, fd);
    int ret = lseek(fd, 0, SEEK_SET);
    HI_EXP_LOGE(ret < 0, "lseek BTN3 file FAIL, err='%s, %d'\n", strerror(errno), errno);
    FdReadAll(fd);
    LOGI("111111111111111111111111111\n");
    LOGI("OnBtn1 down\n");
    if(Key_Function_mod != 1){
        Key_Function_mod = 1; 
    }
    else{
        Key_Function_mod = 0;
    }
    //LOGI("1111111111111111111111111111112222222222222222222222222\n");
}



static int MeterDetectLoad(uintptr_t* model, OsdSet* osds)
{
    HI_S32 ret = 1;
    g_osdsTennis = osds;
    HI_ASSERT(g_osdsTennis);
    g_osd0Tennis = OsdsCreateRgn(g_osdsTennis);
    HI_ASSERT(g_osd0Tennis >= 0);
    *model = 1;

    LOGI("TennisDetectLoad success\n");
    if (GetCfgBool("audio_player:support_audio", true)) {
    ret = SkPairCreate(&g_stmChn);
    //线程创建
    HI_ASSERT(ret == 0);
    if (pthread_create(&g_thrdId, NULL, GetAudioFileName, NULL) < 0) {
        HI_ASSERT(0);
    }
        g_supportAudio = 1;  //如果创建成功就将标志位1
	}
    //按键打开
    gpio1_fd = open("/sys/class/gpio/gpio1/value", O_RDONLY | EFD_CLOEXEC | EFD_NONBLOCK);
	if (gpio1_fd < 0)
	{
		MSG("Failed to open gpio1 !\n");
	}
	fds1[0].fd     = gpio1_fd;
	fds1[0].events = POLLPRI;
	HI_EXP_LOGE(gpio1_fd < 0, "open BTN1 '%s' FAIL, err='%s, %d'\n", "/sys/class/gpio/gpio1/value", strerror(errno), errno);
	gpio2_fd = open("/sys/class/gpio/gpio2/value", O_RDONLY | EFD_CLOEXEC | EFD_NONBLOCK);
	if (gpio2_fd < 0)
	{
		MSG("Failed to open gpio1 !\n");
	}
	fds2[0].fd     = gpio2_fd;
	fds2[0].events = POLLPRI;
    HI_EXP_LOGE(gpio2_fd < 0, "open BTN2 '%s' FAIL, err='%s, %d'\n", "/sys/class/gpio/gpio2/value", strerror(errno), errno);

    //添加回调
    if (gpio1_fd >= 0) {
        FdReadAll(gpio1_fd);
        if (EmAddFd(MainEvtMon(), gpio1_fd, FDE_PRI, OnBtn2, NULL) < 0) {
            assert(0);
        }
    }
    if (gpio2_fd >= 0) {
        FdReadAll(gpio2_fd);
         if (EmAddFd(MainEvtMon(), gpio2_fd, FDE_PRI, OnBtn3, NULL) < 0) {
            assert(0);
        }
    }
	return ret;
}

static int MeterDetectUnload(uintptr_t model)
{
    (void)model;
    OsdsClear(g_osdsTennis);
	if (g_supportAudio == 1) {   //释放掉线程
		SkPairDestroy(&g_stmChn);
		pthread_join(g_thrdId, NULL);
	}
    if (gpio1_fd >= 0) {
        if (EmDelFd(MainEvtMon(), gpio1_fd) < 0) {
            assert(0);
        }
        close(gpio1_fd);
        gpio1_fd = -1;
    }
    if (gpio2_fd >= 0) {
        if (EmDelFd(MainEvtMon(), gpio2_fd) < 0) {
            assert(0);
        }
        close(gpio2_fd);
        gpio2_fd = -1;
    }
    return HI_SUCCESS;
}



HI_S32 yuvFrame2rgb(VIDEO_FRAME_INFO_S *srcFrame, IPC_IMAGE *dstImage)
{
    IVE_HANDLE hIveHandle;
    IVE_SRC_IMAGE_S pstSrc;
    IVE_DST_IMAGE_S pstDst;
    IVE_CSC_CTRL_S stCscCtrl;
    HI_S32 s32Ret = 0;
    stCscCtrl.enMode = IVE_CSC_MODE_PIC_BT709_YUV2RGB; //IVE_CSC_MODE_VIDEO_BT601_YUV2RGB;
    pstSrc.enType = IVE_IMAGE_TYPE_YUV420SP;
    pstSrc.au64VirAddr[0] = srcFrame->stVFrame.u64VirAddr[0];
    pstSrc.au64VirAddr[1] = srcFrame->stVFrame.u64VirAddr[1];
    pstSrc.au64VirAddr[2] = srcFrame->stVFrame.u64VirAddr[2];
 
    pstSrc.au64PhyAddr[0] = srcFrame->stVFrame.u64PhyAddr[0];
    pstSrc.au64PhyAddr[1] = srcFrame->stVFrame.u64PhyAddr[1];
    pstSrc.au64PhyAddr[2] = srcFrame->stVFrame.u64PhyAddr[2];
 
    pstSrc.au32Stride[0] = srcFrame->stVFrame.u32Stride[0];
    pstSrc.au32Stride[1] = srcFrame->stVFrame.u32Stride[1];
    pstSrc.au32Stride[2] = srcFrame->stVFrame.u32Stride[2];
 
    pstSrc.u32Width = srcFrame->stVFrame.u32Width;
    pstSrc.u32Height = srcFrame->stVFrame.u32Height;
 
    pstDst.enType = IVE_IMAGE_TYPE_U8C3_PACKAGE;
    pstDst.u32Width = pstSrc.u32Width;
    pstDst.u32Height = pstSrc.u32Height;
    pstDst.au32Stride[0] = pstSrc.au32Stride[0];
    pstDst.au32Stride[1] = 0;
    pstDst.au32Stride[2] = 0;
   // LOGE("step 1 开始转换~\n");
    s32Ret = HI_MPI_SYS_MmzAlloc_Cached(&pstDst.au64PhyAddr[0], (void **)&pstDst.au64VirAddr[0],
        "User", HI_NULL, pstDst.u32Height*pstDst.au32Stride[0] * 3);
    if (HI_SUCCESS != s32Ret) {       
        HI_MPI_SYS_MmzFree(pstDst.au64PhyAddr[0], (void *)pstDst.au64VirAddr[0]);
        LOGE("HI_MPI_SYS_MmzFree err\n");
        return s32Ret;
    }
   //  LOGE("step 2 000000~\n");
    s32Ret = HI_MPI_SYS_MmzFlushCache(pstDst.au64PhyAddr[0], (void *)pstDst.au64VirAddr[0],
        pstDst.u32Height*pstDst.au32Stride[0]*3);
    if (HI_SUCCESS != s32Ret) {       
        HI_MPI_SYS_MmzFree(pstDst.au64PhyAddr[0], (void *)pstDst.au64VirAddr[0]);
        return s32Ret;
    }
    memset((void *)pstDst.au64VirAddr[0], 0, pstDst.u32Height*pstDst.au32Stride[0]*3);
    HI_BOOL bInstant = HI_TRUE;
    //LOGE("step end -1-1-1-1-1-1-1-1-1-1~\n"); //创建颜色空间转换 的过程中失败了
    // 1 
    s32Ret = HI_MPI_IVE_CSC(&hIveHandle, &pstSrc, &pstDst, &stCscCtrl, bInstant);
    if(HI_SUCCESS != s32Ret) {       
        HI_MPI_SYS_MmzFree(pstDst.au64PhyAddr[0], (void *)pstDst.au64VirAddr[0]);
        LOGE("The caange error num is %d\n",s32Ret);
        return s32Ret;
    }
    //LOGE("step end -1-1-1-1-1-1-1-1-1-1转换结束~\n");
    if (HI_TRUE == bInstant) {
        HI_BOOL bFinish = HI_TRUE;
        HI_BOOL bBlock = HI_TRUE;
        s32Ret = HI_MPI_IVE_Query(hIveHandle, &bFinish, bBlock);
        while (HI_ERR_IVE_QUERY_TIMEOUT == s32Ret) {
            usleep(100);
            s32Ret = HI_MPI_IVE_Query(hIveHandle,&bFinish,bBlock);
        }
    }
     //LOGE("step end 转换结束~\n");
    dstImage->u64PhyAddr = pstDst.au64PhyAddr[0];
    dstImage->u64VirAddr = pstDst.au64VirAddr[0];
    dstImage->u32Width = pstDst.u32Width;
    dstImage->u32Height = pstDst.u32Height;

    return HI_SUCCESS;
}

HI_S32 ipcimg2Mat(VIDEO_FRAME_INFO_S *srcFrame, Mat &dstMat)
{
    HI_U32 w = srcFrame->stVFrame.u32Width;
    HI_U32 h = srcFrame->stVFrame.u32Height;
    int bufLen = w * h * 3;
    HI_U8 *srcRGB = NULL;
    IPC_IMAGE dstImage;
    if (yuvFrame2rgb(srcFrame, &dstImage) != HI_SUCCESS) {
        LOGE("yuvFrame2rgb err\n");
        return HI_FAILURE;
    }
    srcRGB = (HI_U8 *)dstImage.u64VirAddr;
    dstMat.create(h, w, CV_8UC3);
    memcpy(dstMat.data, srcRGB, bufLen * sizeof(HI_U8));
    HI_MPI_SYS_MmzFree(dstImage.u64PhyAddr, (void *)&(dstImage.u64VirAddr));
    return HI_SUCCESS;
}

HI_S32 frame2Mat(IPC_IMAGE *srcFrame, Mat &dstMat)
{
    HI_U32 w = srcFrame->u32Width;
    HI_U32 h = srcFrame->u32Height;
    int bufLen = w * h * 3;
    HI_U8 *srcRGB = NULL;
    //IPC_IMAGE dstImage = *srcFrame;
    // if (yuvFrame2rgb(srcFrame, &dstImage) != HI_SUCCESS) {
    //     LOGE("yuvFrame2rgb err\n");
    //     return HI_FAILURE;
    // }
    srcRGB = (HI_U8 *)srcFrame->u64VirAddr;
    dstMat.create(h, w, CV_8UC3);
    memcpy(dstMat.data, srcRGB, bufLen * sizeof(HI_U8));
    HI_MPI_SYS_MmzFree(srcFrame->u64PhyAddr, (void *)&(srcFrame->u64VirAddr));
    return HI_SUCCESS;
}

/**
    将计算结果打包为resJson.
*/
HI_CHAR* MeterDetectToJson(const RectBox items[], HI_S32 itemNum, int* resBytes)
{
    HI_S32 jsonSize = TINY_BUF_SIZE + itemNum * TINY_BUF_SIZE; // 每个item的打包size为TINY_BUF_SIZE
    HI_CHAR *jsonBuf = (HI_CHAR*)malloc(jsonSize);
    HI_ASSERT(jsonBuf);
    HI_S32 offset = 0;

    offset += snprintf_s(jsonBuf + offset, jsonSize - offset, jsonSize - offset - 1, "[");
    for (HI_S32 i = 0; i < itemNum; i++) {
        const RectBox *item = &items[i];
        offset += snprintf_s(jsonBuf + offset, jsonSize - offset, jsonSize - offset - 1,
            "%s { \"object xmin\": %d, \"ymin\": %d, \"xmax\": %d, \"ymax\": %d }",
            (i == 0 ? "\n  " : ", "), (uint)item->xmin, (uint)item->ymin, (uint)item->xmax, (uint)item->ymax);
        HI_ASSERT(offset < jsonSize);
    }
    offset += snprintf_s(jsonBuf + offset, jsonSize - offset, jsonSize - offset - 1, "]");
    HI_ASSERT(offset < jsonSize);
    
    if (resBytes) {
        *resBytes = offset;
    }
    return jsonBuf;
}

static int ThefileFeatureofimgs(Mat img) {
	char img_path[14] = "";
	int size[25] = { 0 };
	for (int i = 0; i < 23; i++) {
		sprintf(img_path, "./used/%d.xml", i);
		Ptr <SIFT> detector = SIFT::create(450);
		vector<KeyPoint> keypoints_obj;
		Mat descript_info_img, decript_file_img;
		detector->detectAndCompute(img, Mat(), keypoints_obj, descript_info_img);

		FileStorage fs(img_path, FileStorage::READ);
		if (!fs.isOpened()) {
			cout << "Open file Fails" << endl;
			return 1;
		}
		//cout << img_mat_tar << endl;
		fs["MAT"] >> decript_file_img;
		fs.release();

		vector<DMatch> matches;
		vector<vector<DMatch>> knn_matches;
		BFMatcher matcher(NORM_L2); //下面就是knn匹配过程  这块要写成函数反复调用
		matcher.knnMatch(descript_info_img, decript_file_img, knn_matches, 2);
		//舍弃掉大于0.7的匹配
		for (size_t r = 0; r < knn_matches.size(); ++r) {
			if (knn_matches[r][0].distance > 0.7 * knn_matches[r][1].distance) continue;
			matches.push_back(knn_matches[r][0]);
		}
		size[i] = matches.size();
		cout << img_path << ":" << matches.size() << endl;
	}
	int max = 0, max_of_i = 0;
	for (int i = 0; i < 25; i++) {
		max = MAX(max, size[i]);
		if (max == size[i])
			max_of_i = i;
	}
	char out_string[100] = "";
	sprintf(out_string, "This picture is %d.jpg", max_of_i);

	cout << out_string << endl;
	return max_of_i;
}
static int MeterDetectCal(uintptr_t model,
    VIDEO_FRAME_INFO_S *srcFrm, VIDEO_FRAME_INFO_S *dstFrm, HI_CHAR** resJson)
{
    static int64_t start ,end,middle_time; // 定义开始时间及结束时间
    static int8_t send_FLAGE = 0;  //暂时修改为1 当作显示
    (void)model;
    int ret = 0;
    RectBox boxs[TENNIS_OBJ_MAX] = {0};
    HI_CHAR filename[30] = "target/target.jpg";
    
    // LOGD("input srcFrm is %d",strlen(srcFrm));
    if (send_FLAGE == 0){ //第一次进来更新时间
        start = HiClockMs();
        end = HiClockMs();
        middle_time = start;
        if (g_supportAudio == 1) { //第一次进来向管道发送信息
           if (FdWriteMsg(g_stmChn.out, filename, 30*sizeof(HI_CHAR))){
			    LOGE("FdWriteMsg FAIL\n");  
           }
        }
        send_FLAGE = 2; //后面进来就更新end
    }
    else{
        end = HiClockMs();
    }
    //每3秒输出一次LOG
    if((end - middle_time)>3000){
        LOGI("This Fuctionn running ! send_FLAG:%d\n",send_FLAGE);
        LOGI("Button1 cut picture of make!\n");
        LOGI("Running mode is %d\n",Key_Function_mod);
        middle_time =  end ;
    }
    //每1min向管道发送一次数据
    if ((end - start)  >= 10000){
        send_FLAGE = 1;
        start = end;
    }
    Mat image;

    //如果线程创建成功  就像管道发送数据  然后线程读取数据 ------ mode = 2
    if(Key_Function_mod == 2){
        if (g_supportAudio == 1) {
        if(send_FLAGE == 1 ){
            if(ipcimg2Mat(srcFrm, image) != HI_SUCCESS){
                LOGE("Mat fored error\n");
                return ret;
            }
            else {
                imwrite(filename,image);
            }
            LOGI("From pip send missage up up \n");
            if (FdWriteMsg(g_stmChn.out, filename, 30*sizeof(HI_CHAR))) {
			    LOGE("FdWriteMsg FAIL\n"); 
            }
            send_FLAGE = 2;//发送完成后标志位更新
            LOGI("From pip send missage down down \n");
            // LOGI("source srcFrm  Wight is %d\n",pstSrc.u32Width);
            // LOGI("source srcFrm  Height is %d\n",pstSrc.u32Height);
            // LOGI("source srcFrm  PhyAddr is %d\n",pstSrc.u64PhyAddr);
            // LOGI("source srcFrm  VirAddr is %d\n",pstSrc.u64VirAddr);
		}
    }
    }
    //直接拍照处理 mode == 1
    else if (Key_Function_mod == 1 ){
        if (g_supportAudio == 1) {
            if(ipcimg2Mat(srcFrm, image) != HI_SUCCESS){
                LOGE("Mat fored error\n");
                return ret;
            }
            else {
                imwrite(filename,image);
            }
            LOGI("From pip send missage up up \n");
            if (FdWriteMsg(g_stmChn.out, filename, 30*sizeof(HI_CHAR))) {
			    LOGE("FdWriteMsg FAIL\n"); 
            }
        }
    }
    

    //LOGI("**********************************************************************\n");
    return ret;
}

static const AiPlug G_METER_DETECT_ITF = {
    .Prof = MetersDetectProf,
    .Load = MeterDetectLoad,
    .Unload = MeterDetectUnload,
    .Cal = MeterDetectCal,
};

const AiPlug* AiPlugItf(uint32_t* magic)
{
    if (magic) {
        *magic = AI_PLUG_MAGIC;
    }

    return (AiPlug*)&G_METER_DETECT_ITF;
}

