---
title: Android禁用自动休眠和锁屏
date: 2023-10-24 22:29:05
tags:
- Android Framework
- 经验
categories:
- Framework
comments: true

---



> 考古以前放在有道的笔记，仅作记录



#### 休眠

所谓自动休眠，就是指用户长时间内没有跟设备进行交互，然后触发了自动息屏流程。

<!-- more -->

而按power键灭屏或者自动息屏事件都是在**PowerManagerService**中处理的。

具体的流程分析，// TODO

先帖两篇相关的博客：https://www.jianshu.com/p/9241f3a91095 && https://www.cnblogs.com/rainey-forrest/p/13292638.html

总之，在**PowerManagerService**的`updateUserActivitySummaryLocked`方法中，会计算用户没有活动的时间，当这个时间达到用户设置的**timeout**时，就会触发休眠流程。用户设置的休眠时间通过`getScreenOffTimeoutLocked`获取，在`getScreenOffTimeoutLocked`中，发现有一个系统最小的屏幕超时变量`mMinimumScreenOffTimeoutConfig`，这个值在*frameworks/base/core/res/res/values/config.xml*中的`config_minimumScreenOffTimeout`定义，所以，我们实现禁止设备自动休眠的方法为，将这个`config_minimumScreenOffTimeout`设置为尽可能大的值。

```
diff --git a/core/res/res/values/config.xml b/core/res/res/values/config.xml
index 1170fbe..e4ac8d7 100644
--- a/core/res/res/values/config.xml
+++ b/core/res/res/values/config.xml
@@ -2428,7 +2428,7 @@
          This value must be greater than zero, otherwise the device will immediately
          fall asleep again as soon as it is awoken.
     -->
-    <integer name="config_minimumScreenOffTimeout">10000</integer>
+    <integer name="config_minimumScreenOffTimeout">2147483647</integer>
 
     <!-- User activity timeout: Maximum screen dim duration in milliseconds.

```

#### 锁屏

锁屏出现的时机：**LockScreen is shown after reboot or after screen timeout / short press on power**.  // 来自源码`LockPatternUtils.java`中的注释

那么当我们浏览一下`LockPatternUtils.java`时，就会发现其中有一个`isLockScreenDisabled`方法，也就是说，Android系统是支持默认禁用锁屏功能的。

那么当我们看`isLockScreenDisabled`方法的内容时，就会发现：

```
LockPatternUtils.java

/**
 * Determine if LockScreen is disabled for the current user. This is used to decide whether
 * LockScreen is shown after reboot or after screen timeout / short press on power.
 *
 * @return true if lock screen is disabled
 */
@UnsupportedAppUsage
public boolean isLockScreenDisabled(int userId) {                                              
    if (isSecure(userId)) {
        return false;
    }
    boolean disabledByDefault = mContext.getResources().getBoolean(
            com.android.internal.R.bool.config_disableLockscreenByDefault);
    boolean isSystemUser = UserManager.isSplitSystemUser() && userId == UserHandle.USER_SYSTEM;
    UserInfo userInfo = getUserManager().getUserInfo(userId);
    boolean isDemoUser = UserManager.isDeviceInDemoMode(mContext) && userInfo != null
            && userInfo.isDemo();
    return getBoolean(DISABLE_LOCKSCREEN_KEY, false, userId)
            || (disabledByDefault && !isSystemUser)
            || isDemoUser;
}

public final static String DISABLE_LOCKSCREEN_KEY = "lockscreen.disabled";
```

发现有个`disabledByDefault`变量，它在初始化的时候，也是从**config.xml**中获取的初始值，那么理论上可以通过修改`config_disableLockscreenByDefault`的值为true去实现禁用锁屏。

之所以说理论上，是因为我没有去试，这里需要一个有缘人验证可行性。

回过头来看这个方法的return部分，用到了两个逻辑或，也就是说，三个不同的表达式，只要有一个为true，这个方法就会返回true。

`isDemoUser`可能涉及到Android的多用户或者什么特殊模式，不去深究。

所以剩下一个`getBoolean(DISABLE_LOCKSCREEN_KEY, false, userId)`可以探索。

```
private boolean getBoolean(String secureSettingKey, boolean defaultValue, int userId) {
    try {
        return getLockSettings().getBoolean(secureSettingKey, defaultValue, userId);
    } catch (RemoteException re) {
        return defaultValue;
    }    
}        


@UnsupportedAppUsage  
@VisibleForTesting    
public ILockSettings getLockSettings() {                       
    if (mLockSettingsService == null) {
        ILockSettings service = ILockSettings.Stub.asInterface(
                ServiceManager.getService("lock_settings"));
        mLockSettingsService = service;
    }                 
    return mLockSettingsService;
}                     

```

在`getBoolean`方法中，调用了`LockSettingsService.java`的`getBoolean`方法，`getLockSettings()`方法中获取了`ILockSettings`的Binder代理对象，具体实现在`LockSettingsService.java`中。

```
LockSettingsService.java

@Override       
public boolean getBoolean(String key, boolean defaultValue, int userId) {
    checkReadPermission(key, userId);
    String value = getStringUnchecked(key, null, userId);
    return TextUtils.isEmpty(value) ?
            defaultValue : (value.equals("1") || value.equals("true"));
}               

public String getStringUnchecked(String key, String defaultValue, int userId) {
    if (Settings.Secure.LOCK_PATTERN_ENABLED.equals(key)) {
        long ident = Binder.clearCallingIdentity();
        try {      
            return mLockPatternUtils.isLockPatternEnabled(userId) ? "1" : "0";
        } finally {
            Binder.restoreCallingIdentity(ident);
        }          
    }              
                   
    if (userId == USER_FRP) {
        return getFrpStringUnchecked(key);
    }          
               
    if (LockPatternUtils.LEGACY_LOCK_PATTERN_ENABLED.equals(key)) {
        key = Settings.Secure.LOCK_PATTERN_ENABLED;
    }          
               
    return mStorage.readKeyValue(key, defaultValue, userId);
}              

```

看`getBoolean`的具体实现，通过`getStringUnchecked`方法获取返回值。`mStorage`是`LockSettingsStorage`的实例，大概是用来保存一些跟锁屏相关的设置，可能会包括PIN、Password等吧。

看到这里发现是去读某个key的值，这个key的传参为`DISABLE_LOCKSCREEN_KEY`，那么我们可以看一下是在哪里write的`DISABLE_LOCKSCREEN_KEY`的值。

直接在`LockSettingsService.java`里搜索`DISABLE_LOCKSCREEN_KEY`，write相关的共有三处，其中的一处我们发现：

```
public LockSettingsStorage getStorage() {                   
    final LockSettingsStorage storage = new LockSettingsStorage(mContext);
    storage.setDatabaseOnCreateCallback(new LockSettingsStorage.Callback() {
        @Override                                           
        public void initialize(SQLiteDatabase db) {         
            // Get the lockscreen default from a system property, if available
            boolean lockScreenDisable = SystemProperties.getBoolean(
                    "ro.lockscreen.disable.default", false);
            if (lockScreenDisable) {                        
                storage.writeKeyValue(db, LockPatternUtils.DISABLE_LOCKSCREEN_KEY, "1", 0);
            }                                               
        }                                                   
    });                                                     
    return storage;                                         
}                                                           

```

`mStorage`初始化的时候，调用到这个`getStorage`方法，其中会根据一个属性的值，设置`DISABLE_LOCKSCREEN_KEY`的值

也就是说，只要`ro.lockscreen.disable.default`为true，`DISABLE_LOCKSCREEN_KEY`也会设置为1，也就是true，也就是系统默认禁用了锁屏功能。

所以我们只需要将`ro.lockscreen.disable.default`设置为true，就可以禁用锁屏了.

```
diff --git a/tools/buildinfo.sh b/tools/buildinfo.sh
index 9d81dee..e45a441 100755
--- a/tools/buildinfo.sh
+++ b/tools/buildinfo.sh
@@ -8,6 +8,7 @@ echo "ro.build.keys=$BUILD_KEYS"
 if [ -n "$DISPLAY_BUILD_NUMBER" ] ; then
 echo "ro.build.display_build_number=$DISPLAY_BUILD_NUMBER"
 fi
+echo "ro.lockscreen.disable.default=true"
 echo "ro.build.version.incremental=$BUILD_NUMBER"
 echo "ro.build.version.sdk=$PLATFORM_SDK_VERSION"
 echo "ro.build.version.preview_sdk=$PLATFORM_PREVIEW_SDK_VERSION"

```

注意，刷机验证的时候需要把userdata.img刷进去，不然是不会生效的。

> 血与泪的教训：-(
