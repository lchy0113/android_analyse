# smart pointer
> sp<T>

/frameworks/av/camera/Camera.cpp
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

example>
sp<Camera> 


</pr>
