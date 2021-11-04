# camera


## Camera Architecture (legacy)
![](/images/ape_fwk_camera.png)

### app framework

### JNI

### native framework
frameworks/av/camera/Camera.cpp 는 우리가 어플리케이션에서 사용하는 android.hardware.Camera API에 대응되는 native code 이다.
android.hardware.Camera API에서는 JNI를 통해 Camera Service는 IPC binder proxies를 이용하여 접근한다.

### binder IPC proxies
frameworks/av/camera 에 위치한 3가지 바인더 클래스가 존재한다.
- ICameraService
- ICamera
- ICameraClient

### camera service 
안드로이드 프레임워크 framworks/av/services/camera/libcameraservice/CameraService.cpp 에  위치하여 HAL과 통신하는 실제 코드.

### HAL 
실제 카메라 하드웨어를 제어하는 kernel driver를 카메라 서비스가 직접 다루지 않고 중간 층인 HAL을 통해 접근하도록 정의된 표준 인터페이스 이다.

### Kernel driver
실제 카메라 하드웨어와 HAL과 상호 작용을 하는 단계로, 카메라와 드라이버는 YV12, NV21 이미지 형식을 지원해야만 카메라의 미리보기와 비디오 녹화가 가능하다.
