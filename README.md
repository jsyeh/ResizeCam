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
  - (取消)Dragging by Mouse Left Button can move the ResizeCam Window 用滑鼠左鍵拖曳可調位置
  - Scrolling Mouse Wheel can resize the ResizeCam window 用滑鼠滾輪調大小

## Compile Source Code 編譯方式
如果你想要在自己的電腦裡編譯
- Install 安裝 OpenCV 2.1
- 我使用 CodeBlocks 17.12 (附 MinGW) 有對應的專案檔 ResizeCam.cbp
  - 裡面預先把 OpenCV 2.1 的 include 及 lib 目錄手動設好了
  - 也預先把 cv210 cxcore210 highgui210 對應的檔案也設定好了
- 也可使用 Visual Studio 2022 來編譯程式, 附上對應的 ResizeCam.vcxproj

## Source Code 程式碼
```cpp
///源由: 疫情期間線上上課/錄影時, 想在Windows桌面秀出老師的WebCam畫面
///      原本使用呂聰賢老師推薦的 ViewDirector Lite 但後來更新會卡住
///      就自己利用 OpenCV 2.1 開發簡單的工具程式 (2022-11-16)
///用法: 點擊執行檔, 會把第一個 WebCam 秀在畫面左上角
///      (取消)Mouse 左鍵可移動位置 (2023-09-22 取消此功能, 可Drag標題列移動視窗)
///      Mouse Wheel 捲動可縮放大小
///      Esc 鍵可離開(並釋放資源) (按右上角X不會關閉哦!)
///      按 - (減號) 可調成原本大小
#pragma comment(linker, "/SUBSYSTEM:windows /ENTRY:mainCRTStartup")
/// 上面這行, 可以在 Visual Studio 裡, 設定 linker 使用 Windows Subsystem 不要秀出 Console

#include <opencv/highgui.h>
#include <stdio.h>
int CapX = 0, CapY = 0;
double CapW = 1920, CapH = 1080; ///預設解析度 1920x1080
char CapName[] = "ResizeCam";
HWND myself = NULL;
double scale = 1.0;
void mouse(int event, int x, int y, int flag, void* param)
{
    /*static int oldX = 0, oldY = 0, pressed = 0;//(取消)Mouse 左鍵可移動位置 (2023-09-22 取消此功能, 可Drag標題列移動視窗)
    if (event == CV_EVENT_LBUTTONDOWN) {
        oldX = x;
        oldY = y;
        pressed = 1;
    }
    else if (pressed == 1 && event == CV_EVENT_MOUSEMOVE) {
        printf("x-oldX: %d y-oldY:%d\n", x - oldX, y - oldY); //若not active, x, y值在 cvMoveWindow()後, 會變超大, 很奇怪
        CapX += x - oldX;
        CapY += y - oldY;
        oldX = x;
        oldY = y;
        cvMoveWindow(CapName, CapX, CapY);
        printf("CapX:%d CapY:%d %d %d\n", CapX, CapY, GetSystemMetrics(SM_CXSCREEN), GetSystemMetrics(SM_CYSCREEN));
    }
    else if (event == CV_EVENT_LBUTTONUP) {
        pressed = 0;
    }*/
}
int wndcallback(HWND hwnd, UINT uMsg, WPARAM wParam, LPARAM lParam, int* was_processed)
{
    switch (uMsg) {
    case WM_MOUSEWHEEL:
        int zDelta = GET_WHEEL_DELTA_WPARAM(wParam);
        if (zDelta > 0) scale *= 1.1;
        if (zDelta < 0) scale *= 0.9;
        cvResizeWindow(CapName, (int)(CapW * scale), (int)(CapH * scale));
    }
    return 0;
}
int main()
{
    CvCapture* cap = cvCreateCameraCapture(0);
    cvSetCaptureProperty(cap, CV_CAP_PROP_FRAME_WIDTH, 1024);
    cvSetCaptureProperty(cap, CV_CAP_PROP_FRAME_HEIGHT, 768);

    CapW = cvGetCaptureProperty(cap, CV_CAP_PROP_FRAME_WIDTH);
    CapH = cvGetCaptureProperty(cap, CV_CAP_PROP_FRAME_HEIGHT);
    cvNamedWindow(CapName, CV_WINDOW_NORMAL);
    cvResizeWindow(CapName, (int)CapW, (int)CapH);
    cvMoveWindow(CapName, CapX, CapY);
    cvSetMouseCallback(CapName, mouse); ///處理Mouse Drag移動
    cvSetPostprocessFuncWin32(wndcallback); ///處理Mouse Wheel縮放大小
    myself = GetForegroundWindow();

    while (1) {
        IplImage* frame = cvQueryFrame(cap);
        cvShowImage(CapName, frame);
        int key = cvWaitKey(35);
        if (key == 27) break; ///按下 Esc 鍵可退出
        if (key == '-') { ///按下 - 減號, 可調回 1920x1080的大小
            scale = 1.0;
            cvResizeWindow(CapName, (int)(CapW * scale), (int)(CapH * scale));
        }
    }
    cvReleaseCapture(&cap); ///釋放 Camera 資源很重要
}
```
