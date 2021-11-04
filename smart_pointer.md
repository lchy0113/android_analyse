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
```
//#define LOG_NDEBUG 0
#define LOG_TAG "Camera"
#include <utils/Log.h>
#include <utils/threads.h>
#include <utils/String16.h>
#include <binder/IPCThreadState.h>
#include <binder/IServiceManager.h>
#include <binder/IMemory.h>
```
__#include <binder/IMemory.h>__ 이동

_/frameworks/native/include/binder/IMemory.h__
```
#ifndef ANDROID_IMEMORY_H
#define ANDROID_IMEMORY_H

#include <stdint.h>
#include <sys/types.h>
#include <sys/mman.h>

#include <utils/RefBase.h>
#include <utils/Errors.h>
#include <binder/IInterface.h>
```
__#include <utils/RefBase.h>__ 이동

_/system/core/include/utils/RefBase.h__
```
#ifndef ANDROID_REF_BASE_H
#define ANDROID_REF_BASE_H

#include <atomic>

#include <stdint.h>
#include <sys/types.h>
#include <stdlib.h>
#include <string.h>

// LightRefBase used to be declared in this header, so we have to include it
#include <utils/LightRefBase.h>

#include <utils/StrongPointer.h>
#include <utils/TypeHelpers.h>
```
__#include <utils/StrongPointer.h>__ 이동


_/system/core/include/utils/StrongPointer.h_
```
template<typename T>
class sp {
public:
    inline sp() : m_ptr(0) { }

    sp(T* other);  // NOLINT(implicit)
    sp(const sp<T>& other);
    sp(sp<T>&& other);
    template<typename U> sp(U* other);  // NOLINT(implicit)
    template<typename U> sp(const sp<U>& other);  // NOLINT(implicit)
    template<typename U> sp(sp<U>&& other);  // NOLINT(implicit)

    ~sp();

    // Assignment

    sp& operator = (T* other);
    sp& operator = (const sp<T>& other);
    sp& operator = (sp<T>&& other);

    template<typename U> sp& operator = (const sp<U>& other);
    template<typename U> sp& operator = (sp<U>&& other);
    template<typename U> sp& operator = (U* other);

    //! Special optimization for use by ProcessState (and nobody else).
    void force_set(T* other);

    // Reset

    void clear();

    // Accessors

    inline  T&      operator* () const  { return *m_ptr; }
    inline  T*      operator-> () const { return m_ptr;  }
    inline  T*      get() const         { return m_ptr; }

    // Operators

    COMPARE(==)
    COMPARE(!=)
    COMPARE(>)
    COMPARE(<)
    COMPARE(<=)
    COMPARE(>=)

private:    
    template<typename Y> friend class sp;
    template<typename Y> friend class wp;
    void set_pointer(T* ptr);
    T* m_ptr;
};
```
__sp<T>__ 가 구현된 __StrongPointer__ 가 정의되어 있다.

안드로이드 소스코드에 사용되고 있는 sp<T> 는 스마트포인터(SmartPointer)를 말하는 것이며, 보통 struct나 int같이 C에서 기본적으로 제공되고 있는 것들은 기호 *을 통해 간단하게 포인터로 사용하는 것이 가능합니다.
__sp<T>__ 는 이를 객체인 Class를 포인터처럼 사용할 수 있도록 하기 위해 사용되는 도구라고 설명드릴 수 있겠습니다. 
이를 사용하면 사용자는 포인터로 지정된 객체를 실제 객체를 다루듯이 사용하실 수 있다는 장점을 가지고 있습니다. 

Strong Pointer를 쓰는 가장 중요한 이유는 포인터 객체의 할당과 삭제를 직접 제어할 수 있다는 것입니다. 
java와 같이 Gabage Collector 기능이 있는 경우 객체를 할당 해준 후 사용하지 않으면 알아서 할당에서 사라지지만  C/C++ 의 경우 해당 기능이 존재 하지 않습니다. 
그렇기 때문에 __sp<T>__ 를 활용해 메모리를 관리해 준다면 좀 더 쾌적한 프로그램을 만들 수 있을 것입니다.

그렇다면 여기서 잠시 코드 내부를 살펴보도록 하겠습니다.

```
template<typename T>
sp<T>::sp(T* other)
        : m_ptr(other) {
    if (other)
        other->incStrong(this);
}
```
객체의 스마트포인터가 할당된 경우입니다. 기존에 해당 객체가 존재할 경우, incStrong()를 통해 참조 갯수를 늘립니다.

```
template<typename T>
sp<T>::~sp() {
    if (m_ptr)
        m_ptr->decStrong(this);
}
```
객체의 스마트포인터 하나가 할당이 해제된 경우입니다. decStrong()을 통해 참조 갯수를 줄입니다.

```
template<typename T>
void sp<T>::clear() {
    if (m_ptr) {
        m_ptr->decStrong(this);
        m_ptr = 0;
    }
}
```
객체 자체의 할당을 해제합니다. 해당포인터를 0으로 하여 할당을 해제합니다.

```
template<typename T>
class sp {
public:
    inline sp() : m_ptr(0) { }

	...
        // Accessors

    inline  T&      operator* () const  { return *m_ptr; }
    inline  T*      operator-> () const { return m_ptr;  }
    inline  T*      get() const         { return m_ptr; }

	...

private:    
    template<typename Y> friend class sp;
    template<typename Y> friend class wp;
    void set_pointer(T* ptr);
    T* m_ptr;
};
```
앞에서도 제시되었던 SmartPointer인 sp 클래스 내부의 모습입니다.  Accessors 주석 아래에 작성된 부분은 Smart Pointer에 연결된 포인터에 접근에 관하여 다루고 있는 모습입니다.

