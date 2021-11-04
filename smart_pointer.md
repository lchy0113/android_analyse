# smart pointer

_/frameworks/av/camera/Camera.cpp_
```
// construct a camera client from an existing camera remote
sp<Camera> Camera::create(const sp<::android::hardware::ICamera>& camera)
{
     ALOGV("create");
     if (camera == 0) {
         ALOGE("camera remote is a NULL pointer");
         return 0;
     }

    sp<Camera> c = new Camera(-1);
    if (camera->connect(c) == NO_ERROR) {
        c->mStatus = NO_ERROR;
        c->mCamera = camera;
        IInterface::asBinder(camera)->linkToDeath(c);
        return c;
    }
    return 0;
}


```

> example>
```
sp<Camera> 
```

---


_/frameworks/av/camera/Camera.cpp_
```cpp
//#define LOG_NDEBUG 0
#define LOG_TAG "Camera"
#include <utils/Log.h>
#include <utils/threads.h>
#include <utils/String16.h>
#include <binder/IPCThreadState.h>
#include <binder/IServiceManager.h>
#include <binder/IMemory.h>
```
__#include <binder/IMemory.h>__
