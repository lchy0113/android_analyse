# Add service



1. system service 를 추가한다. 아래의 코드를 생성한다.  

_frameworks/base/services/core/java/com/android/server/GpioService.java_
```
package com.android.server;
import android.os.IGpioService;
import android.util.DebugUtils;
import android.util.Slog;

public class GpioService extends IGpioService.Stub
{
    private static final String TAG = "GpioService";
	private static final boolean DEBUG = true;

    /* call native c function to access hardware */
    public int gpioWrite(int gpio, int value) throws android.os.RemoteException
    {
		if (DEBUG) {
			Slog.d(TAG, "gpioWrite");
		}
        return native_gpioWrite(gpio, value);
    }
    
    public int gpioRead(int gpio) throws android.os.RemoteException
    {
		if (DEBUG) {
			Slog.d(TAG, "gpioRead");
		}
        return native_gpioRead(gpio);
    }
    
    public int gpioDirection(int gpio, int direction, int value) throws android.os.RemoteException
    {
		if (DEBUG) {
			Slog.d(TAG, "gpioDirection");
		}
        return native_gpioDirection(gpio, direction, value);
    }
    
    public int gpioRegKeyEvent(int gpio) throws android.os.RemoteException
    {
		if (DEBUG) {
			Slog.d(TAG, "gpioRegKeyEvent");
		}
        return native_gpioRegKeyEvent(gpio);
    }
    
    public int gpioUnregKeyEvent(int gpio) throws android.os.RemoteException
    {
		if (DEBUG) {
			Slog.d(TAG, "gpioUnregKeyEvent");
		}
        return native_gpioUnregKeyEvent(gpio);
    }
    
    public int gpioGetNumber() throws android.os.RemoteException
    {
		if (DEBUG) {
			Slog.d(TAG, "gpioGetNumber");
		}
        return native_gpioGetNumber();
    }
    
    public GpioService()
    {
		if (DEBUG) {
			Slog.d(TAG, "GpioService");
		}
        native_gpioOpen();
    }

    public static native int native_gpioOpen();
    public static native void native_gpioClose();
    public static native int native_gpioWrite(int gpio, int value);
    public static native int native_gpioRead(int gpio);
    public static native int native_gpioDirection(int gpio, int direction, int value);
    public static native int native_gpioRegKeyEvent(int gpio);
    public static native int native_gpioUnregKeyEvent(int gpio);
    public static native int native_gpioGetNumber();
}
```

2. 추가로 만든 서비스를 등록한다.
_frameworks/base/services/java/com/android/server/SystemServer.java_

```
...

			Slog.i(TAG, "Gpio Service");
			GpioService gpio = new GpioService();
			ServiceManager.addService("gpio", gpio);

...
```

3. aidl을 추가하여 애플리케이션에서 추가된 서비스에 접근할 수 있는 인터페이스를 만든다.

_frameworks/base/core/java/android/os/IGpioService.aidl_
```
package android.os;

/** {@hide} */
interface IGpioService
{
	/**
	 *	GPIO output control
	 */
	int gpioWrite(int gpio, int value) = 0;
	/**
	 *	Read GPIO data
	 */
	int gpioRead(int gpio) = 1;
	/**
	 *	GPIO direction control
	 */
	int gpioDirection(int gpio, int direction, int value) = 2;
	/**
	 *	Registration GPIO interrupt (key event) in Input mode.
	 */
	int gpioRegKeyEvent(int gpio) = 3;
	/**
	 *	Unregister input mode GPIO interrupt (key event).
	 */
	int gpioUnregKeyEvent(int gpio) = 4;
	/**
	 *	Check the number of available GPIOs.
	 */
	int gpioGetNumber() = 5;
}
```

4. 추가한 aidl 파일을 등록한다.

_frameworks/base/Android.mk_
```
	core/java/android/os/IGpioService.aidl \
```

5. build.

6. build 완료 후, 새로 수정된 프레임워크 라이브러리를 추출한다. 라이브러리의 경로는 아래와 같다

_out/target/common/obj/JAVA_LIBRARIES_
```
out/target/common/obj/JAVA_LIBRARIES$ ls framework_intermediates/
anno     classes2.dex  classes.dex.toc         classes.jack  classes.jar.toc  dotdot              jack-rsc                   javalib.jar  link_type  lowpan  packages  src       telephony  with-local
classes  classes.dex   classes-full-debug.jar  classes.jar   core             jack_res_jar_flags  jack-rsc.java-source-list  keystore     location   media   proto     telecomm  wifi
```

자바 라이브러리들을 확인 할 수 있다.추가한 시스템 서비스는 __framework_intermediates__ 경로 내에 있다.
디렉토리 내에 classes.jar 파일이 있는데 이 안에 생성한 서비스가 들어있다. 

```
android/os/IGpioService.class
android/os/IGpioService$Stub$Proxy.class
android/os/IGpioService$Stub.class 
```

class 파일이 내부에 있는것을 확인 할 수 있다. 이것이 안드로이드 기기 전원이  ON상태인 동안 백그라운드에서 기기가 OFF될때 까지 계속 실행된다.

