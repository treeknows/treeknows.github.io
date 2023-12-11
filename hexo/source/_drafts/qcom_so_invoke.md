### 高通平台现状

高通把QVR相关的bin和so都放在了vendor目录下，如`qvrservice`、`libqvrservice.so`等，同样的，对于`qvrservice`涉及到的client端的实现也放在了vendor目录。

高通qvrservice cilent端涉及到的so有

- `libqvrservice_client.qti.so`
- `libqvrcamera_client.qti.so`

但是有一点很奇怪的地方，**没有在设备任何目录下的`public.libraries*.txt`中发现把以上的两个client端so添加进去**，但是在基线代码中查找可以在 *device/qcom*目录下找到添加了client so的`public.libraries*.txt`，不确定是因为相关的txt文件没有被打包到img还是有一些别的未知的处理。

#### 适配新变化

同时，为了适配Android12的安全策略，高通在SXR SDK 4.0.6版本以及后续的OpenXR SDK新增了一些so和头文件

- `libqxrcoreclient.so`
- `libqxrcamclient.so`
- `libqxrsplitclient.so`
- `QXRCoreClient.h`
- `QXRCamClient.h`

因此，原本的一些调用方式也发生了一些变化。

1. 6DOF初始化相关的API调用由原本的`QVRServiceClient_Create()`变为了`QXRCoreClient_Create(JavaVM* javaVM, jobject context)`，

2. camera相关的初始化API由`QVRCameraClient_Create()`变为了` QXRCamClient_Create(JavaVM* javaVM, jobject context)`

以上QXR开头的API由`QXRCoreClient.h`和`QXRCamClient.h`提供，且头文件中非内联函数，其具体的实现包含在对应的so中。以及，还会涉及到`qxr_client.aar`中的一些调用。

#### QXR Client

上面提到的`qxr_client.aar`其实是作为**QXR Core Client**，会通过某种IPC的方式连接到**QXR Core Service**（Maybe AIDL）。**QXR Core Service**是运行在设备上的服务，其可能对应的实现应该位于`xrcbservice`，预置在*/system_ext/priv-app/xrcbservice/xrcbservice.apk*。

相关一些开发注意事项如下：

**Note 1**： 可以先调用旧的API，再调用新的API。旧的API在Android S以前的版本可以正常拿到句柄，在Android S上会返回`NULL`：

```C
qvrservice_client_helper_t *client = QVRServiceClient_Create();   // legacy function
if (NULL == client) {
	client = QXRCoreClient_Create(gAppContext->javaVm, gAppContext->javaActivityObject);
}
```

**Note 2**：**QXR Core Service**必须知道Client端的包名。默认的包名是"com.qualcomm.qti.spaces.services"。如果是其他应用，需要在调用API时传入应用的包名：

```C
client = QXRCoreClient_Create(javaVm, javaActivityObject, "com.example.xrservice");
```

**Note 3**：**QXR Core Client**连接了**QXR Core Service**，其创建和销毁的动作都应该执行在同一条线程。

**Note 4**：在应用的`AndroidManifest.xml`中应该声明以下的权限：

```xml
<uses-permission android:name="com.qualcomm.qti.qxr.QXRServiceClientPermission" />
```

处理好上述的四点内容，应该就可以成功拿到`libqvrservice_clietn.qti.so`里的句柄了，就可以正常调用6dof的一些API了。

### Ark项目架构

Ark项目拥有一套类似高通的框架，同样将相关的bin和so放在了vendor目录，目前在设备中找到的部分如下：

- `vxrservice`
- `libopenxr_service_client.so`
- `libopenxr_vxrcamera_client.so`
- `libvxr_eyetracking_plugin.so`
- `libvxr_slam.so`

一些关键的so如`libvxrservice.so`等并没有在设备的系统库目录下发现，那么**vxrservice是否能正常运行呢？**

#### Ark’s Core Client

Ark项目同样实现了一个`libopenxr_core_client.so`，并且似乎也预置到了*vendor/lib*等目录下。但是类比高通的话，该so是不需要预置到系统中的，而是应该在应用端打包使用。

目前`libopenxr_core_client.so`的实现只包含`CoreClient.cpp`，其实现为：

```c++
vxrservice_client_helper_t* VXRCoreClient_Create(JavaVM* javaVM, jobject context,
    const char* servicePackageName)
{
    if (!javaVM || !servicePackageName) {
    }

    gAppContext.javaActivityObject = context;

    return VXRServiceClient_Create();
}

```

目前看来似乎远远不够。

#### 研发背景

上层正在研发的Runtime/Conroller相关的功能都需要通过**vxrservice**拿到Head Pose或Camera Frame，因此需要能够加载到vxrservice client端的`libopenxr_service_client.so`等，与vxrservice通信。

理想中的调用流程应该为：APP(Runtime/Controller)集成`libopenxr_core_client.so`和`VXRCoreClient.h`等头文件到应用内，通过`VXRCoreClient_Create(JavaVM* javaVM, jobject context)`拿到Client端的句柄，然后执行后续的生命周期调用等操作。

#### 当前问题

实现上述理想操作时遇到的问题有二：

1. 应用导入`libopenxr_core_client.so`等之后，运行过程中会出现dlopen “libc++.so”等类似的报错。

   >10-20 03:24:53.558  4168  4168 E AndroidRuntime: java.lang.UnsatisfiedLinkError: dlopen failed: library "libc++.so" not found: needed by /data/app/~~FBcDGn0j27hdW4VE_JBaRw==/com.thundercomm.test2-bw_0aCus55NcGLE3px598Q==/base.apk!/lib/arm64-v8a/libopenxr_core_client.so in namespace classloader-namespace
   >10-20 03:24:53.558  4168  4168 E AndroidRuntime: 	at java.lang.Runtime.loadLibrary0(Runtime.java:1077)
   >10-20 03:24:53.558  4168  4168 E AndroidRuntime: 	at java.lang.Runtime.loadLibrary0(Runtime.java:998)
   >10-20 03:24:53.558  4168  4168 E AndroidRuntime: 	at java.lang.System.loadLibrary(System.java:1656)
   >10-20 03:24:53.558  4168  4168 E AndroidRuntime: 	at com.thundercomm.test2.MainActivity.<clinit>(MainActivity.java:15)
   >···

2. 临时解决上述问题后，还会遇到无法获得Client端句柄的问题

#### 问题验证

在法贵之前demo的基础上作如下验证：

1. 使用高通的`libqxrcoreclient.so`等验证高通qvr的相关功能

   验证过程中首先遇到的第一个问题就是，同样会报dlopen相关的错误。

   >10-20 03:39:07.767  4466  4466 E AndroidRuntime: java.lang.UnsatisfiedLinkError: dlopen failed: library "libc++_shared.so" not found: needed by /data/app/~~XJU-Dx9WpJF4RxyfFXeaNg==/com.thundercomm.test2-shVaiceY9QeOV4wWfExefg==/base.apk!/lib/arm64-v8a/libqxrcamclient.so in namespace classloader-namespace
   >10-20 03:39:07.767  4466  4466 E AndroidRuntime: 	at java.lang.Runtime.loadLibrary0(Runtime.java:1077)
   >10-20 03:39:07.767  4466  4466 E AndroidRuntime: 	at java.lang.Runtime.loadLibrary0(Runtime.java:998)
   >10-20 03:39:07.767  4466  4466 E AndroidRuntime: 	at java.lang.System.loadLibrary(System.java:1656)
   >10-20 03:39:07.767  4466  4466 E AndroidRuntime: 	at com.thundercomm.test2.MainActivity.<clinit>(MainActivity.java:15)

   与上述问题1中报错类似但不一样，且通过修改`build.gradle`文件中的配置可以解决该问题

   ```xml
               cmake {
                   arguments "-DANDROID_STL=c++_shared", "-DANDROID_ARM_NEON=TRUE", "-DCMAKE_ANDROID_ARCH_ABI=arm64-v8a"
                   abiFilters "arm64-v8a"
               }
   ```

   解决完上述的问题之后，测试通过，可以正常拿到句柄。

   测试代码：

   ```
   qvrservice_client_helper_t *clientHelper = QXRCoreClient_Create(jni_android_info.vm,
                                                                       jni_android_info.context,
                                                                       "com.thundercomm.test2");
       if (clientHelper != nullptr) {
           JLOGI("wanglin clienthelper != null");
           ret = QVRServiceClient_SetTrackingMode(clientHelper, TRACKING_MODE_POSITIONAL);
           if (ret == 0) {
               JLOGI("wanglin: QVRServiceClient_SetTrackingMode success!");
           } else {
               JLOGI("wanglin: QVRServiceClient_SetTrackingMode FAILED!");
           }
       } else {
           JLOGI("wanglin clienthelper == null");
       }
   ```

   输出结果：

   > 10-20 04:10:41.309  6545  6545 I LFG_VXRAPITEST: wanglin clienthelper != null
   > 10-20 04:10:41.309  6545  6545 I LFG_VXRAPITEST: wanglin: QVRServiceClient_SetTrackingMode success!

2. 使用Ark项目的`libopenxr_core_client.so`进行验证

   直接运行会dlopen报错失败，日志如下：

   > 10-20 05:49:33.187  9027  9027 E AndroidRuntime: java.lang.UnsatisfiedLinkError: dlopen failed: library "libc++.so" not found: needed by /data/app/~~GLMn94kJknY0pq4k__JLTA==/com.thundercomm.test2-bBKlQDgABNOrAkQ6jJt2hQ==/base.apk!/lib/arm64-v8a/libopenxr_core_client.so in namespace classloader-namespace

   针对此问题，一种可行的解决办法是，修改*/system/etc/public.libraries.txt*，增加数行内容

   ```
   libc++.so
   libutils.so
   libcutils.so
   libvndksupport.so
   libprocessgroup.so
   libui.so
   ```

   设备重启后重新运行，问题解决。

   此时依旧无法成功dlopen `libopenxr_service_client.so`，需要手动将*vendor/lib*下的`libopenxr_*.so`移动到*system/lib*目录下，并在*system/etc/public.libraries.txt*中添加内容

   ```
   ...
   libopenxr_service_client.so
   libopenxr_vxrcamera_client.so
   ```

   重启之后可以dlopen success。

   但同样会出现拿不到句柄的现象：

   > logcat如下：
   >
   > 10-20 03:25:08.786  3999  3999 I VXRServiceClient: dlsym success
   > 10-20 03:25:08.786  3999  3999 I VXRServiceClient: wanglin: clientHelper->client != nullptr
   > 10-20 03:25:08.786  3999  3999 I VXRServiceClient: wanglin: clientHelper->client->clientOps->Create != nullptr
   > 10-20 03:25:08.787  3999  3999 I VXRServiceClient: wanglin: !clientHelper->clientHandle
   > 10-20 03:25:08.788  3999  3999 I LFG_VXRAPITEST: wanglin clienthelper == null
   > 10-20 03:25:08.788  3999  3999 I LFG_VXRAPITEST: wanglin: camClientHelper == null

   ```
   ...
   CLOGI("dlsym success");
       clientHelper->client = vxrServiceClient();
       if (clientHelper->client != nullptr) {
           CLOGI("wanglin: clientHelper->client != nullptr");
       }
       if (clientHelper->client->clientOps->Create != nullptr) {
           CLOGI("wanglin: clientHelper->client->clientOps->Create != nullptr");
       }
       clientHelper->clientHandle = clientHelper->client->clientOps->Create();
       if (!clientHelper->clientHandle) {
           dlclose(clientHelper->libHandle);
           free(clientHelper);
           CLOGI("wanglin: !clientHelper->clientHandle");
           return NULL;
       }
       CLOGI("client create success");
       return clientHelper;
   ```

   根据日志显示的结果，`Create()`方法的返回值为`NULL`，也就是说，目前能够成功调用`libopenxr_service_client.so`，但是无法返回有效的句柄，因此下一步需要从`libopenxr_service_client.so`的实现来分析了。

   `libopenxr_service_client.so`的实现在`ServiceClient.cpp`中，通过加日志发现在初始化client时失败：

   > 10-20 06:44:22.426  7497  7497 I VXR_ServiceClient: VxrServiceHTClient start
   > 10-20 06:44:22.426  7497  7497 E VXR_Socket: connect: ERROR connecting to socket (Permission denied)
   > 10-20 06:44:22.426  7497  7497 E VXR_ConClient: init: ERROR connect to server                       
   > 10-20 06:44:22.426  7497  7497 E VXR_ServiceClient: ERROR init client

   此时可能的原因有二：

   - 既有的实现代码有问题，无法连接到`vxrservice`
   - `libopenxr_service_client.so`移到system目录下后，无法正常与vendor目录下的`vxrservice`建立socket通信

   解决办法如下：

   - 修改`/dev/socket/vxr*`的权限为777
   - `setenforce 0`

   修改socket file权限和禁用SELinux要同时进行

   > SELinux报错：
   >
   > 10-20 08:03:29.971  8631  8631 I undercomm.test2: type=1400 audit(0.0:5828): avc: denied { write } for name="vxrservice_vndr" dev="tmpfs" ino=35088 scontext=u:r:untrusted_app:s0:c147,c256,c512,c768 tcontext=u:object_r:socket_device:s0 tclass=sock_file permissive=1 app=com.thundercomm.test2
   >
   > 10-20 08:03:29.971  8631  8631 I undercomm.test2: type=1400 audit(0.0:5829): avc: denied { connectto } for path="/dev/socket/vxrservice_vndr" scontext=u:r:untrusted_app:s0:c147,c256,c512,c768 tcontext=u:r:vxrservice:s0 tclass=unix_stream_socket permissive=1 app=com.thundercomm.test2
   >
   > 10-20 08:03:29.971  8631  8631 I undercomm.test2: type=1400 audit(0.0:5830): avc: denied { use } for path="/dmabuf:dmabuf322" dev="dmabuf" ino=35147 scontext=u:r:untrusted_app:s0:c147,c256,c512,c768 tcontext=u:r:vxrservice:s0 tclass=fd permissive=1 app=com.thundercomm.test2

   测试结果日志输出如下：

   > 10-20 08:03:29.977  8631  8631 I VXRServiceClient: client create success
   > 10-20 08:03:29.979  8631  8631 I LFG_VXRAPITEST: wanglin clienthelper != null
   > 10-20 08:03:29.979  8631  8631 D VXR_ServiceClient: setHeadTrackingMode: 2 success
   > 10-20 08:03:29.979  8631  8631 I LFG_VXRAPITEST: wanglin: VXRServiceClient_SetTrackingMode success!
   > 10-20 08:03:29.979  8631  8631 I LFG_VXRAPITEST: wanglin: camClientHelper != null

   到此workaround的调研就顺利结束了。

#### so调用问题workaround

需要做的工作总结如下：

- 修改*/system/etc/public.libraries.txt*，解决dlopen libc++问题
- 两个client so移动到system目录并添加白名单
- 修改socket file的权限
- SELinux临时设为permissive

做完以上四点就可以在应用端导入core client so以及相关的头文件进行开发了

#### core client so dlopen failed

前面提到过，使用demo应用通过`libopenxr_core_client.so`获取vxrservice client句柄时会遇到dlopen libc++的报错，临时的解决办法为把报错的so添加*system/etc/public.libraries.txt*白名单中。在so调用问题找到解决办法之后，回过头来分析此问题。

##### 原因

使用的demo应用是一个native应用，也就意味着有native的实现，通常native部分的代码会被打包成so库，通过`loadLibrary`方法加载

```
static {
    System.loadLibrary("test2");
}
```

除了native实现打包的so，demo中还要用到`libopenxr_core_client.so`，于是在`CMakeLists.txt`中做了link操作

```cmake
add_library(openxr_core_client
        SHARED
        IMPORTED)
set_target_properties(openxr_core_client
        PROPERTIES IMPORTED_LOCATION
        ${CMAKE_LIBRARY_OUTPUT_DIRECTORY}/libopenxr_core_client.so)
        
target_link_libraries( # Specifies the target library.
        test2
        openxr_core_client
        # Links the target library to the log library
        # included in the NDK.
        ${log-lib})
```

于是，应用启动后加载相关so时，除了`libtest2.so`，还会去加载`libopenxr_core_client.so`，而client so依赖`libc++`动态库，`libc++`正好又属于非公开的NDK库，总结起来就是：应用使用(link)了一个依赖私有NDK库(libc++)的第三方库(libopenxr_core_client)

##### 解决

1. 高通的做法

   使用`readelf -dW`命令可以发现，高通的`libqxrcoreclient.so`依赖的是`libc++_shared.so`，通过修改demo应用的构建脚本，可以使`libc++_shared.so`一起打包在应用中，从而优先加载应用安装目录下的lib目录下的so库，不必去system目录下加载

   ```
               cmake {
                   arguments "-DANDROID_STL=c++_shared", "-DANDROID_ARM_NEON=TRUE", "-DCMAKE_ANDROID_ARCH_ABI=arm64-v8a"
                   abiFilters "arm64-v8a"
               }
   ```

2. 可以尝试手动放置一个`libc++.so`，和应用一起打包---未尝试

3. TC的做法

   本想仿照高通，修改bp文件，使client so依赖`libc++_shared.so`，奈何Android Runtime STL中只包含了**共享库`libc++.so`**和**静态库`libc++_static.so`**（源码位于*external/libcxx*内），并且`libopenxr_core_client.so`也并非直接依赖`libc++`，而可能是间接依赖，因此无法继续进行下去。这也侧面反映了高通`libqxrcoreclient.so`等库的编译可能是通过NDK-build的方式编出来的。

   最终的修改方式为依赖`libc++`的静态版本

   ```
   diff --git a/Android.bp b/Android.bp
   index 9d86941..7d0e922 100644
   --- a/Android.bp
   +++ b/Android.bp
   @@ -282,6 +282,7 @@ cc_library_shared {
        shared_libs: [
            "liblog",
        ],
   +    stl: "libc++_static",
        proprietary: true,
    }
   
   ```

#### client so预置vendor目录方案

当`libopenxr_service_client.so`位于vendor目录下时，运行demo应用就无法拿到相关句柄。

> VXRServiceClient: dlopen: libopenxr_service_client.so
> VXRServiceClient: can't open libopenxr_service_client.so, error: dlopen failed: library "libopenxr_service_client.so" not found

从日志中可以看到，应用在dlopen so时，并没有找到，因此无法打开。

所以当前的关键问题就在于：为什么把so移到vendor目录下之后dlopen就找不到相关的so了？

基于当前的问题做了多种尝试：

- 将so添加到*vendor/etc/public.libraries.txt*中，验证结果依旧失败，报错和上述相同

- 依据 [谷歌官方](https://source.android.com/docs/core/permissions/namespaces_libraries?hl=zh-cn#adding-additional-native-libraries)的方法，尝试性的关闭了SELinux（从logcat或dmesg看不到任何avc denied的报错），发现依旧报错，但是报错内容有变化

  > 11-04 10:24:52.697  4073  4073 I VXRServiceClient: wanglin: dlopen: libopenxr_service_client.so
  > 11-04 10:24:52.698  4073  4073 I VXRServiceClient: wanglin: can't open libopenxr_service_client.so, error: dlopen failed: library "libui.so" not found: needed by /vendor/lib64/libopenxr_service_client.so i
  > n namespace sphal
  > 11-04 10:24:52.699  4073  4073 I LFG_VXRAPITEST: wanglin clienthelper == null
  > 11-04 10:24:52.699  4073  4073 I LFG_VXRAPITEST: wanglin: camClientHelper == null

- 上述报错表明，`libopenxr_service_client.so`可以找到了，但是其依赖的`libui.so`在`namespace sphal`找不到，基于这个发现，又做了新的验证    ----Vendor binary 测试能否正常调用

- 在设备里搜索`libui.so`发现，vendor目录下没有该so，只分布于system或apex目录下

  > ./system/lib/libui.so
  > ./system/apex/com.android.vndk.current/lib/libui.so
  > ./system/apex/com.android.vndk.current/lib64/libui.so
  > ./system/apex/com.android.media.swcodec/lib64/libui.so
  > ./system/lib64/libui.so
  > ./system_ext/apex/com.android.vndk.v30/lib/libui.so
  > ./system_ext/apex/com.android.vndk.v30/lib64/libui.so
  > ./system_ext/apex/com.android.vndk.v31/lib/libui.so
  > ./system_ext/apex/com.android.vndk.v31/lib64/libui.so
  > ./apex/com.android.vndk.v31/lib/libui.so
  > ./apex/com.android.vndk.v31/lib64/libui.so
  > ./apex/com.android.vndk.v30/lib/libui.so
  > ./apex/com.android.vndk.v30/lib64/libui.so
  > ./apex/com.android.vndk.v32/lib/libui.so
  > ./apex/com.android.vndk.v32/lib64/libui.so
  > ./apex/com.android.media.swcodec/lib64/libui.so

  于是尝试复制`libui.so`到vendor目录下，结果无法开机，有其他系统so依赖报错

- 于是反向验证，想办法使`libopenxr_service_client.so`不依赖`libui.so`，于是在Android.bp里注释掉了相关的依赖，同时注释了代码中的调用，发现对`libopenxr_service_client.so`的调用成功了

  > VXRServiceClient: wanglin: dlopen: libopenxr_service_client.so
  > VXRServiceClient: dlsym success
  > VXRServiceClient: wanglin: clientHelper->client != nullptr
  > ...
  > VXRServiceClient: client create success
  > LFG_VXRAPITEST: wanglin clienthelper != null
  > VXR_ServiceClient: setHeadTrackingMode: 2 success
  > LFG_VXRAPITEST: wanglin: VXRServiceClient_SetTrackingMode success!
  > LFG_VXRAPITEST: wanglin: camClientHelper == null

  在这一步同样需要给socket文件777权限，但是到此基本就验证成功了，但是可行的方案还是有点不清晰





qvrservice依赖`libqvrcamera_client.qti.so`，删除so会导致qvrservice起不来，但是`libqvrservice_client.qti.so`似乎不会影响qvrservice

> 10-20 03:24:09.230  2829  2829 F linker  : CANNOT LINK EXECUTABLE "/vendor/bin/hw/qvrservice": library "libqvrcamera_client.qti.so" not found: needed by /vendor/lib64/libqvrservice.so in namespace (default)                                                                                                                                                                                                

























---

静态库和动态库的区别：0

用标准C++库`libc++`来说，可以分为`libc++.so`和`libc++_shared.so`两种

- `libc++.so`是`libc++`库的静态版本。当一个程序依赖于`libc++.so`时，它会将`libc++`的代码和功能链接到程序的可执行文件中。这意味着程序在运行时不需要依赖外部的`libc++`库文件，因为所有必要的代码已经被静态地包含在可执行文件中。这样的好处是，程序可以在任何没有`libc++`库文件的系统上运行，因为它已经包含了所有必要的代码。
- `libc++_shared.so`是`libc++`库的共享版本。与静态版本不同，共享版本的库文件并不包含实际的代码，而是包含了指向实际代码的链接。当一个程序依赖于`libc++_shared.so`时，它会在运行时动态加载该库文件，并将程序与实际的`libc++`代码进行链接。这意味着程序在运行时需要访问外部的`libc++`库文件，因此必须确保系统中存在相应的共享库文件。



对于Android NDK开发来说，无论是静态库还是动态库，`libc++`都是用NDK里的发布版本打包在应用里：

- 动态库直接在apk里带上`libc++_shared.so`，也就是说，**apk文件解压缩后，lib目录下会有`libc++_shared.so`存在**
- 静态库把程序需要的STL的代码直接打到应用程序或其所用的so库里，也就是说，apk文件解压后没有`libc++_shared.so`，因为**`libc++`相关的代码已经被打包到`libtest.so`了，也就是native应用的so**



CMake支持的STL类型：

```
// Android-Common.cmake
set(_ANDROID_STL_TYPES
    none
    system
    c++_static
    c++_shared
    gabi++_static
    gabi++_shared
    gnustl_static
    gnustl_shared
    stlport_static
    stlport_shared
    )
```





我们已经知道了在Android源码内提供了多种STL支持，其中Android Runtime STL也就是libc++.so是AOSP默认缺省使用的STL，但在NDK开发中是无法使用该STL的，以当前最新的NDK版本r25b为例，提供的STL设定选项包括三种：libc++\system\none，在编译阶段使用名称指代时分为c++_shared,c++_static,none,或system，前两者都是libc++.

初次一看，相比大家都比较疑惑，libc++是不是就对应libc++.so呢，当然不是了。这里例举这三个选项所对应的具体的STL标准库：

libc++选项：包含共享库libc++shared.so，静态库libc++static.a.

system选项：对应于libstdc++.so.

none选项：不提供STL支持.

事实上在NDK r18之前，NDK中还提供了gnustl和stlport，在r18之后则不再提供，此时libc++.so属于AOSP源码编译时缺省的默认STL库。

在使用NDK编译工具链时，我们需要对STL进行设定，这里将结合不同的编译系统来说明如何进行设定。

https://coderfan.net/en/stl-support-in-android.html



疑问：

1. 源码里可以看到有预置`libqvr*.so`的地方，在**public.libraries*.txt**中，但是实际上刷完版本adb shell去看所有的txt，没有发现预置
2. BP文件中的shared_libs和static_libs有什么区别
3. 放到system/etc/public.libraries.txt中的libopenxr_service_client.so就必须预置到system/lib64目录下
4. 10-20 05:17:23.766   614   614 I servicemanager: Found vendor.qti.hardware.qxr.IQXRCoreService/default in device VINTF manifest.
5. 适用于HAL的AIDL https://source.android.com/docs/core/architecture/aidl/aidl-hals?hl=zh-cn

