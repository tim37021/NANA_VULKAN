# Vulkan Guide for Nana Users

Suppose you know everything about Vulkan. The following instruction is written for windows. But it is almost the same for other systems

1. Enable two extensions when creating instance. VK_KHR_SURFACE_EXTENSION_NAME and VK_KHR_WIN32_SURFACE_EXTENSION_NAME
2. Create vulkan surface for windows with vkCreateWin32SurfaceKHR. Pass nana::form::native_handle() to HWND parameter. for C++ wrapper vk::Instance::createWin32SurfaceKHR.
3. Check if the queue family supports presentation on the surface with vkGetPhysicalDeviceSurfaceSupportKHR. for C++ wrapper vk::PhysicalDevice::getSurfaceSupportKHR member function
4. Move your render stuff to other thread.(It will save your time..) Nana will block on nana::exec call. Or you can use draw_through event. Please refer to OpenGL example in this article https://sourceforge.net/p/nanapro/blog/2015/01/tour-of-nana-10/

Currently c++ demo is finished. I will finish C version demo soon :-) Note that include/lib path must be change to build the project.

I've marked and comment on every important code. simply search "**MODIFIED BY TIM**"

## Threading in Vulkan is delightful. WATCH!
```cpp
Demo demo;

#include <thread>
#include <atomic>
void render_thread(Demo &demo, std::atomic<bool> &should_stop) 
{
	while (!should_stop.load()) {
		demo.run();
	}
}

int WINAPI WinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance, LPSTR pCmdLine, int nCmdShow) {
    // ...
    // skipped
    demo.init(argc, argv);
    // skipped
    // ....

    demo.connection = hInstance;
	memcpy((char *)demo.name, (const char *)L"cube", APP_NAME_STR_LEN);
    demo.create_window();
    demo.init_vk_swapchain();

    demo.prepare();

    std::atomic<bool> should_stop;
    should_stop.store(false);
    std::thread t(render_thread, std::ref(demo), std::ref(should_stop));

    nana::exec();
    should_stop.store(true);
    t.join();
    demo.cleanup();

    return (int)msg.wParam;
}
```

## Known issues

1. Changing window caption frequently makes vulkan surface blink.