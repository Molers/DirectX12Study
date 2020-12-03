# DirectX12Study
DX12学习

参考路线：[想学习DirectX12，请问大家有这方面的什么好书推荐?](https://www.zhihu.com/question/34795960/answer/661657868)

# [DirectX 12 - Braynzar Soft Tutorials](https://www.braynzarsoft.net/viewtutorial/q16390-04-directx-12-braynzar-soft-tutorials)

## 1.Setting up DirectX 12 for Visual Studio 2015

网站上设定的是2015但是我就直接就用2019了，然后C:\Program Files (x86)\Windows Kits\10路径下已经有Windows10 kit开发套装了，可能是安装Visual studio和window10的时候自动安装的？

选择windows桌面应用程序，在Debug里把x86移除，按教程所说x64程序可以使用使用1TB的内存而且比x86更快，第一次听说。

### 1.1 VC++包含目录设置

VC++目录里有着Window10SDK的默认包含目录$(WindowsSDK_IncludePath)也有shared和um目录

Windows Kits->10->Include->{version}->shared 

Windows Kits->10->Include->{version}->um

Shared文件夹包含有Microsoft DirectX Graphics Infrastructure (DXGI)，dxgi.h ,dxgi1_2.h等用于swapchain

um文件夹 包含有"d3d12.h, "d3dcompiler.h and "d2d12sdklayers.h" (used for debugging) 

### 1.2 VC++库目录设置

$(WindowsSDK_LibraryPath_x64)下面包括的有

C:\Program Files (x86)\Windows Kits\10\lib\10.0.18362.0\um\x64

### 1.3 链接器->输入

这个倒是没自带，加上去。

```
d3d12.lib
dxgi.lib
d3dcompiler.lib
```

### 1.4 DirectX Math

微软的数学库，按照作者的意思是新建一个头文件d3dx12.h然后复制粘贴过来，然后就找不到D3D12_RESOURCE_DESC1标识符，？是版本太低了吗看了看Windows 10 SDK 版本 2004 (10.0.19041.0)的d3dx12.h和Windows 10 SDK 版本 1903 (10.0.18362.1)是不一样的...[DirectX-Graphics-Samples/Libraries/D3DX12 at master · microsoft/DirectX-Graphics-Samples (github.com)](https://github.com/Microsoft/DirectX-Graphics-Samples/tree/master/Libraries/D3DX12)

[DirectX-Graphics-Samples](https://github.com/microsoft/DirectX-Graphics-Samples)

[DirectX-Graphics-Samples/d3dx12.h at master · microsoft/DirectX-Graphics-Samples (github.com)](https://github.com/Microsoft/DirectX-Graphics-Samples/blob/master/Samples/Desktop/D3D12HelloWorld/src/HelloTriangle/d3dx12.h)

诶，电脑里虽然有旧版的d3dx12.h也能凑合着用，之后再改吧。

### 1.5 stdafx.h

The stdafx.h file is a common filename for a "grouping" of all headers and code that doesn't change often. 

也就是说C++带的这个文件是吧所有的头文件分组在一起

但是在VS创建的时候好像附带有Framework.h  功能和stdafx.h功能一样

## 2.Creating a Window

```c++
// Handle to the window
HWND hwnd = NULL;

// name of the window (not the title)
LPCTSTR WindowName = L"DX12App";

// title of the window
LPCTSTR WindowTitle = L"DX12Window";
// width and height of the window
int Width = 800;
int Height = 600;

// is window full screen?
bool FullScreen = false;

// create a window
bool InitializeWindow(HINSTANCE hInstance,
    int ShowWnd,
    int width, int height,
    bool fullscreen);

// main application loop
void mainloop();

// callback function for windows messages
LRESULT CALLBACK WndProc(HWND hWnd,
    UINT msg,
    WPARAM wParam,
    LPARAM lParam);
//main
int WINAPI WinMain(HINSTANCE hInstance,    //Main windows function
    HINSTANCE hPrevInstance,
    LPSTR lpCmdLine,
    int nShowCmd)
{
    // create the window
    if (!InitializeWindow(hInstance, nShowCmd, Width, Height, FullScreen))
    {
        MessageBox(0, L"Window Initialization - Failed",
            L"Error", MB_OK);
        return 0;
    }

    // start the main loop
    mainloop();

    return 0;
}

//InitializeWindow
bool InitializeWindow(HINSTANCE hInstance,int ShowWnd,int width, int height,bool fullscreen)
{   
    //是否全屏
    if (fullscreen)
    {
        HMONITOR hmon = MonitorFromWindow(hwnd,
            MONITOR_DEFAULTTONEAREST);
        MONITORINFO mi = { sizeof(mi) };
        GetMonitorInfo(hmon, &mi);

        width = mi.rcMonitor.right - mi.rcMonitor.left;
        height = mi.rcMonitor.bottom - mi.rcMonitor.top;
    }

    WNDCLASSEX wc;//describe our window.

    wc.cbSize = sizeof(WNDCLASSEX);
    wc.style = CS_HREDRAW | CS_VREDRAW;
    wc.lpfnWndProc = WndProc;
    wc.cbClsExtra = NULL;
    wc.cbWndExtra = NULL;
    wc.hInstance = hInstance;
    wc.hIcon = LoadIcon(NULL, IDI_APPLICATION);
    wc.hCursor = LoadCursor(NULL, IDC_ARROW);
    wc.hbrBackground = (HBRUSH)(COLOR_WINDOW + 2);
    wc.lpszMenuName = NULL;
    wc.lpszClassName = WindowName;
    wc.hIconSm = LoadIcon(NULL, IDI_APPLICATION);

    if (!RegisterClassEx(&wc))//register the class
    {
        MessageBox(NULL, L"Error registering class",
            L"Error", MB_OK | MB_ICONERROR);
        return false;
    }

    //create our window with the class we just registered by calling the CreateWindowEX function.
    hwnd = CreateWindowEx(NULL,WindowName,WindowTitle,WS_OVERLAPPEDWINDOW,CW_USEDEFAULT, 
        CW_USEDEFAULT,width, height,NULL,NULL,hInstance,NULL);

    if (!hwnd)
    {
        MessageBox(NULL, L"Error creating window",
            L"Error", MB_OK | MB_ICONERROR);
        return false;
    }
    //If we want full screen, we will need to remove the window style.
    if (fullscreen)
    {
        SetWindowLong(hwnd, GWL_STYLE, 0);
    }
    //Finally we show the window and update the window, then return true.
    ShowWindow(hwnd, ShowWnd);
    UpdateWindow(hwnd);

    return true;
}
//If we did not find a message, we run our game code and continue the loop.
void mainloop() {
    MSG msg;
    ZeroMemory(&msg, sizeof(MSG));

    while (true)
    {
        if (PeekMessage(&msg, NULL, 0, 0, PM_REMOVE))
        {
            if (msg.message == WM_QUIT)
                break;

            TranslateMessage(&msg);
            DispatchMessage(&msg);
        }
        else {
            // run game code
        }
    }
}
LRESULT CALLBACK WndProc(HWND hWnd, UINT message, WPARAM wParam, LPARAM lParam)//windows消息的回调函数
{
    switch (message)
    {
    case WM_KEYDOWN:
        if (wParam == VK_ESCAPE) {
            if (MessageBox(0, L"Are you sure you want to exit?",
                L"Really?", MB_YESNO | MB_ICONQUESTION) == IDYES)
                DestroyWindow(hwnd);
        }
        return 0;
    case WM_DESTROY://esc键
        PostQuitMessage(0);
        return 0;
    default:
        return DefWindowProc(hWnd, message, wParam, lParam);
    }
  
}

```

