# ResizeCam
A Windows webcam tool which can Resize and Move by mouse wheel and drag

可利用滾輪調大小、利用滑鼠滾輪調大小、利用拖曳調位置

![ResizeCam](https://user-images.githubusercontent.com/3252557/202056721-5310f61e-1f55-41d6-ae8f-c43bd0499a5c.png)

## Usage 使用方法
- Install 安裝方法:
  - Download 下載 ResizeCam.zip
  - Unzip 解壓縮
- 使用方法
  - Run 執行 ResizeCam.exe
  - Dragging by Mouse Left Button can move the ResizeCam Window 用滑鼠左鍵拖曳可調位置
  - Scrolling Mouse Wheel can resize the ResizeCam window 用滑鼠滾輪調大小

## Compile Source Code 編譯方式
如果你想要在自己的電腦裡編譯
- Install 安裝 OpenCV 2.1
- 我使用 CodeBlocks 17.12 (附 MinGW) 有對應的專案檔 ResizeCam.cbp
  - 裡面預先把 OpenCV 2.1 的 include 及 lib 目錄手動設好了
  - 也預先把 cv210 cxcore210 highgui210 對應的檔案也設定好了

## Source Code 程式碼
```cpp
///源由: 疫情期間線上上課/錄影時, 想在Windows桌面秀出老師的WebCam畫面
///      原本使用呂聰賢老師推薦的 ViewDirector Lite 但後來更新會卡住
///      就自己利用 OpenCV 2.1 開發簡單的工具程式 (2022-11-16)
///用法: 點擊執行檔, 會把第一個 WebCam 秀在畫面左上角
///      Mouse 左鍵可移動位置
///      Mouse Wheel 捲動可縮放大小
///      Esc 鍵可離開(並釋放資源) (按右上角X不會關閉哦!)
#include <opencv/highgui.h>
int CapX=0, CapY=0, CapW=640, CapH=480;
char CapName[]="ResizeCam";
float scale=1.0;
void mouse(int event, int x, int y, int flag, void* param)
{
    static int oldX=0, oldY=0, pressed=0;
    if(event==CV_EVENT_LBUTTONDOWN){
        oldX = x;
        oldY = y;
        pressed = 1;
    }else if(pressed==1 && event==CV_EVENT_MOUSEMOVE){
        CapX += x-oldX;
        CapY += y-oldY;
        cvMoveWindow(CapName, CapX, CapY);
    }else if(event==CV_EVENT_LBUTTONUP){
        pressed = 0;
    }
}
int wndcallback(HWND hwnd, UINT uMsg, WPARAM wParam, LPARAM lParam, int* was_processed)
{
    switch (uMsg){
    case WM_MOUSEWHEEL:
        int zDelta = GET_WHEEL_DELTA_WPARAM(wParam);
        if(zDelta>0) scale *= 1.1;
        if(zDelta<0) scale *= 0.9;
        cvResizeWindow(CapName, CapW*scale, CapH*scale);
    }
}
int main()
{
    CvCapture * cap = cvCreateCameraCapture(0);
    CapW = cvGetCaptureProperty(cap, CV_CAP_PROP_FRAME_WIDTH);
    CapH = cvGetCaptureProperty(cap, CV_CAP_PROP_FRAME_HEIGHT);

    cvNamedWindow(CapName, CV_WINDOW_NORMAL);
    cvResizeWindow(CapName, CapW, CapH);
    cvMoveWindow(CapName, CapX, CapY);
    cvSetMouseCallback(CapName, mouse); ///處理Mouse Drag移動
    cvSetPostprocessFuncWin32(wndcallback); ///處理Mouse Wheel縮放大小

    while(1){
        IplImage * frame = cvQueryFrame(cap);
        cvShowImage(CapName, frame);
        int key = cvWaitKey(35);
        if(key==27) break; ///按下 Esc 鍵可退出
    }
    cvReleaseCapture(&cap); ///釋放 Camera 資源很重要
}
```
