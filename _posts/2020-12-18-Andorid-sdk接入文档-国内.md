---
title: Android自主接入文档（国内）
author: wuxiaowei
date: 2021-05-06 13:00:00 +0800
categories: [Blogging, Tutorial]
tags: [Android,国内]
pin: true
---

# Android系统适配与打包相关配置

## AndroidX配置

如果创建的项目时支持AndroidX不用修改。如果是旧项目，请使用 Android Studio 3.2 及更高版本，菜单栏中依次选择 Refactor > Migrate to AndroidX，即可将现有项目迁移到 AndroidX。

+ 修改 gradle.properties，加入以下代码块

```
android.useAndroidX=true
android.enableJetifier=true 

```

Android 插件会通过重写现有第三方库的二进制文件，自动将这些库迁移为使用 AndroidX，**特别需要注意原生广告**。

还有疑问可以看google的[**迁移到 AndroidX**](https://developer.android.com/jetpack/androidx/migrate)文档


## MultiDex 配置

### minSdkVersion >=21

系统默认情况下会启用 MultiDex，并且您不需要 MultiDex 支持库。

### minSdkVersion <=20

您必须使用 MultiDex 支持库并对应用项目进行以下修改：

### 修改模块级 build.gradle 文件

启用 MultiDex，并将 MultiDex 库添加为依赖项，如下所示：

```groovy
    android {
        defaultConfig {
            ...
            multiDexEnabled true
        }
        ...
    }

    dependencies {
        implementation 'androidx.multidex:multidex:2.0.1'
    }

```

### 继承 Application 类，执行以下某项操作：

+ 重写 attachBaseContext() 方法并调用 MultiDex.install(this) 以启用 MultiDex：
  
  ```java
    public class MyApplication extends Application {
        @Override
        protected void attachBaseContext(Context base) {
            super.attachBaseContext(base);
            MultiDex.install(this);
        }
    }
  ```
+ 请修改manifest文件以设置 **application** 标记中的 android:name，替换成你的Application全类名
  
  ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <manifest xmlns:android="http://schemas.android.com/apk/res/android"
        package="你的包名">
        <application
                android:name="com.xxx.MyApplication" >
            ...
        </application>
    </manifest>

  ```
[更多MultiDex相关文档](https://developer.android.com/studio/build/multidex)    


## Android 9以上适配

+ 在AndroidManifest中新增以下配置
  
```xml
<application>
    
    <uses-library android:name="org.apache.http.legacy" android:required="false"/>
    
</application>
```

+ 兼容部分第三方广告SDK存在Http请求 在AndroidManifest的application的标签中增加：android:networkSecurityConfig 的配置：

```xml
<application
    android:networkSecurityConfig="@xml/network_security_config"
    android:fullBackupContent="@xml/custom_backup_rule"
    tools:replace="android:fullBackupContent"
    >
    
</application>
```
+ 其中在项目的res/xml文件夹新增network_security_config.xml，内容如下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <base-config cleartextTrafficPermitted="true" />
</network-security-config>
```



# SDK基础配置

## 项目根目录的build.gradle增加以下内容

```groovy
buildscript {

    repositories {
        maven { url 'https://dl.bintray.com/umsdk/release' }
    }

    allprojects {
        repositories {
            
            maven {
                url 'https://repo.rdc.aliyun.com/repository/74503-release-qNEqtU/'
                credentials {
                    username 'p5gXfa'
                    password 'wKY0RHNSH3'
                }
            }
            maven { url 'https://dl.bintray.com/umsdk/release' }
            maven {
                url 'https://artifact.bytedance.com/repository/pangle'
            }
        
        }
    }
}
```

# 广告配置

## 引入广告依赖

app module的build.gradle 添加以下内容

```groovy

defaultConfig {
    ndk { abiFilters  'armeabi-v7a','arm64-v8a'}
}

packagingOptions {
    doNotStrip "*/armeabi-v7a/*.so"
    doNotStrip "*/arm64-v8a/*.so"
}

repositories {
    flatDir {
        dirs 'libs'
    }
}
dependencies {
    //删除旧版的引入
    //implementation 'com.eyu:eyulibrary:xxx'
    
    //sdk核心库（必须）
    implementation 'com.eyu.opensdk:core-ch:1.9.0'
    
    //国内通常使用穿山甲
    implementation 'com.eyu.opensdk.ad.mediation:pangle-ch-adapter:3.5.0.8.34'
    
    //按需求引入广告平台

    //穿山甲
    //implementation 'com.eyu.opensdk.ad.mediation:pangle-ch-adapter:3.5.0.8.34'


    //广点通
    //implementation 'com.eyu.opensdk.ad.mediation:gdt-adapter:4.360.1230.29'

     //topon
    //implementation 'com.eyu.opensdk.ad.mediation:topon-adapter:5.7.25.28'

        //mtg
    //implementation 'com.eyu.opensdk.ad.mediation:mtg-ch-adapter:13.0.41.28'
}
```

## 穿山甲

无

## 广点通

无

## topon

下载将[所需的topon的库](https://docs.toponad.com/#/zh-cn/android/download/package),拷贝到工程目录的libs下，然后在gradle中的android下加入以下内容

```groovy
dependencies {
    api fileTree(include: ['*.jar','*.aar'], dir: 'topon_libs')
}

```

按需引入以下配置到manifest文件，根据你所引入的库来决定引入哪些

```xml
    <!-- 穿山甲 start -->
        <provider
            android:name="com.bytedance.sdk.openadsdk.multipro.TTMultiProvider"
            android:authorities="${applicationId}.TTMultiProvider"
            android:exported="false" />
        <provider
            android:name="com.bytedance.sdk.openadsdk.TTFileProvider"
            android:authorities="${applicationId}.TTFileProvider"
            android:exported="false"
            android:grantUriPermissions="true">
            <meta-data
                android:name="android.support.FILE_PROVIDER_PATHS"
                android:resource="@xml/anythink_bk_tt_file_path" />
        </provider>
    <!-- 穿山甲 end -->
        <!-- source file: china/network_sdk/baidu/AndroidManifest.xml -->
        <!-- 非信息广告必须添加 -->
        <activity
            android:name="com.baidu.mobads.AppActivity"
            android:configChanges="screenSize|keyboard|keyboardHidden|orientation"
            android:theme="@android:style/Theme.Translucent.NoTitleBar"/>

        <provider
            android:name="com.baidu.mobads.openad.BdFileProvider"
            android:authorities="${applicationId}.bd.provider"
            android:exported="false"
            android:grantUriPermissions="true">
            <meta-data
                android:name="android.support.FILE_PROVIDER_PATHS"
                android:resource="@xml/anythink_bk_baidu_file_path" />
        </provider>
        <!-- source file: china/network_sdk/sigmob/AndroidManifest.xml -->

        <!-- targetSDKVersion >= 24时才需要添加这个provider。
        provider的authorities属性的值为${applicationId}.sigprovider -->
        <provider
            android:name="com.sigmob.sdk.SigmobFileProvider"
            android:authorities="${applicationId}.sigprovider"
            android:exported="false"
            android:grantUriPermissions="true">
            <meta-data
                android:name="android.support.FILE_PROVIDER_PATHS"
                android:resource="@xml/anythink_bk_sigmob_file_path"/>
        </provider>

        <!-- source file: china/network_sdk/gdt/AndroidManifest.xml -->

        <provider
            android:name="com.qq.e.comm.GDTFileProvider"
            android:authorities="${applicationId}.gdt.fileprovider"
            android:exported="false"
            android:grantUriPermissions="true">
            <meta-data
                android:name="android.support.FILE_PROVIDER_PATHS"
                android:resource="@xml/anythink_bk_gdt_file_path" />
        </provider>
```

## mtg

无

# 旧版本升级注意

新版的包名改了，将代码中的报错import删除，重新引入，删除SdkHelper相关代码


# SDK使用

## SDK初始化

请在 Application 中的onCreate方法中初始化sdk，添加配置信息

```java
//
InitializerBuilderImpl builder = new InitializerBuilderImpl();
builder.setDebugMode(BuildConfig.DEBUG);
//appsflyer配置
//builder.configAppsFlyer(“appkey”);

//热云
//builder.configTrackingIO("appKey");

//友盟
//builder.initUmeng("appKey","channel");

//数数的统计初始化
//builder.configThinkData("appid");

SdkCompat.getInstance().init(Application, builder);

```

## 权限申请

在你的Activity中调用

```java
SdkCompat.getInstance().requestPermissions(this);
```

## 权限回调

在你的Activity中调用

```java
@Override
public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
    super.onRequestPermissionsResult(requestCode, permissions, grantResults);
    SdkCompat.getInstance().onRequestPermissionsResult(this, requestCode, permissions, grantResults);
}
```

## 广告配置

广告配置有三个文件，ad_setting.json，ad_cache_setting.json，ad_key_setting.json，and_ad_group_setting.json（此文件不一定有）实际给到的名字前面可能会有and_前缀，将它们放到**res/raw**目录下，将后面会用到。


## 广告初始化

在你的Activity中初始化广告

```java
AdConfig adConfig = new AdConfig();

//前面提到的ad_setting.json
adConfig.setAdPlaceConfigResource(this, R.raw.ad_setting);
//adConfig.setAdPlaceConfigStr

//前面提到的ad_key_setting.json
adConfig.setAdKeyConfigResource(this, R.raw.ad_key_setting);
//adConfig.setAdKeyConfigStr()

//前面提到的ad_cache_setting.json
adConfig.setAdGroupConfigResource(this, R.raw.ad_cache_setting);
//adConfig.setAdGroupConfigStr

adConfig.setAdGroupChildStr(this, R.raw.and_ad_group_setting);


//穿山甲
bundle = new Bundle();
bundle.putString(PlatformExtras.COMMON_APP_ID, "");
bundle.putString(PangleExtras.APP_NAME, "");
adConfig.addPlatformConfig(AdPlatform.PANGLE, bundle);

//mtg
//bundle = new Bundle();
//bundle.putString(PlatformExtras.COMMON_APP_ID, "");
//bundle.putString(PlatformExtras.COMMON_APP_KEY, "");
//adConfig.addPlatformConfig(AdPlatform.MTG, bundle);

//TOPON
//bundle = new Bundle();
//bundle.putString(PlatformExtras.COMMON_APP_ID, "");
//bundle.putString(PlatformExtras.COMMON_APP_KEY, "");
//adConfig.addPlatformConfig(AdPlatform.TOPON, bundle);

//广点通
//bundle = new Bundle();
//bundle.putString(PlatformExtras.COMMON_APP_ID, "");
//adConfig.addPlatformConfig(AdPlatform.GDT, bundle);

EyuAdManager.getInstance().config(MainActivity.this, adConfig, new EyuAdsListener() {

    @Override
    public void onAdReward(AdFormat type, String placeId) {
        //激励视频获得奖励，type.getLable()获得广告类型，placeId为广告位置
    }

    @Override
    public void onAdLoaded(AdFormat type, String placeId) {
        //广告加载成功
    }

    @Override
    public void onAdShowed(AdFormat type, String placeId) {
        //广告展示
    }

    @Override
    public void onAdClosed(AdFormat type, String placeId) {
        //广告被关闭
    }

    @Override
    public void onAdClicked(AdFormat type, String placeId) {
        //广告被点击
    }

    @Override
    public void onDefaultNativeAdClicked() {

    }

    @Override
    public void onAdLoadFailed(AdFormat type, String placeId, String key, int code) {
        //广告加载失败，如果没有广告展示，可以过滤日志onAdLoadFailed查看广告加载失败原因
    }

    @Override
    public void onImpression(AdFormat type, String placeId) {
        //广告展示，同onAdShowed
    }

});
   
```

## 广告使用示例

调用EyuAdManager.getInstance().show(AdFormat,"adPlaceId")方法展示广告，传入的**adPlaceId** 为前面提到的**ad_setting.json**中的id

### 判断广告是否有可用的广告

```java
EyuAdManager.getInstance().isAdLoaded(AdFormat,"adPlaceId")
```

### 展示激励视频

```java
EyuAdManager.getInstance().show(AdFormat.REWARDED, Activity,"adPlaceId");
```

### 展示插屏

```java
EyuAdManager.getInstance().show(AdFormat.INTERSTITIAL, Activity,"adPlaceId");
```

### 展示banner

需要传入一个ViewGroup，这个group是用来放banner的
```java
EyuAdManager.getInstance().show(AdFormat.BANNER, Activity,ViewGroup,"adPlaceId");
```

### 展示原生广告

需要传入一个ViewGroup，这个group是用来放native的
```java
EyuAdManager.getInstance().show(AdFormat.NATIVE, Activity,ViewGroup,"adPlaceId");

```


# 事件埋点

## 基本数据埋点

调用下面的方法，事件会上传到umeng和Appsflyer，**不会上传到数数** 如果要上传到数数，请额外调用数数的方法
```java
//事件不带参数
EventHelper.getInstance().logEvent("事件名称");
//事件带map参数
EventHelper.getInstance().logEventWithParamsMap("事件名称",new HashMap<String, Object>());
//事件带json字符串参数
EventHelper.getInstance().logEventWithJsonParams("事件名称","json");
```

## 数数的埋点

**你可能会用到这个，看给你的配置中有没有数数的配置**
EventHelper对数数sdk的方法只是做了一层简单的封装，并没有做任何处理，所以先看下[数数的文档](https://docs.thinkingdata.cn/ta-manual/latest/installation/installation_menu/client_sdk/android_sdk_installation/android_sdk_installation.html#%E4%B8%89%E3%80%81%E5%8F%91%E9%80%81%E4%BA%8B%E4%BB%B6)，了解每个方法的含义


> EventHelper.getInstance().track("");

```java

EventHelper.getInstance().track("事件名称");

EventHelper.getInstance().track("事件名称",JSONObject);//参数

//更多方法
void track(String var1);

void track(String var1, JSONObject var2);

void timeEvent(String var1);

void login(String var1);

void logout();

void identify(String var1);

void user_set(JSONObject var1);

void user_setOnce(JSONObject var1);

void user_add(JSONObject var1);

void user_append(JSONObject var1);

void user_add(String var1, Number var2);

void user_delete();

void user_unset(String... var1);

void setSuperProperties(JSONObject var1);

void trackAppInstall();

void trackFirst(String var1, JSONObject var2);

void trackUpdate(String var1, JSONObject var2, String var3);
```

# 常见问题

+ sdk下载失败？  
  检查build.gradle配置是否添加，如果添加好后还是不能加载成功，请检查网络是否连通

+ 没有广告展示？  
  1.请检查广告配置是否正确配置，如果配置好了，在Android studio的日志打印那里过滤onAdLoadFailed，有错误码打印，将错误码提供给支持

+ 出现构建错误:unexpected element ＜queries＞ found in ＜manifest＞
  修改根目录build.gradle文件中的Android Gradle 插件版本
  当前使用->建议升级版本<br>
  3.3.x->3.3.3<br>
  3.4.x->3.4.3<br>
  3.5.x->3.5.4<br>
  3.6.x->3.6.4<br>
  4.0.x->4.0.1<br>

  举个例子，如果您正在使用 4.0.0 版本的 Android Gradle 插件，就可以在项目级别的 build.gradle 文件中将相关依赖升级到上图中对应的版本。

```groovy
buildscript {
    dependencies {

        //
        // classpath 'com.android.tools.build:gradle:4.0.0'
        classpath 'com.android.tools.build:gradle:4.0.1'
    }
}
```




# 示例工程 

[示例工程](https://github.com/EyugameQy/EyuLibrary-android/tree/master/app_ch_new)，建议先仔细看一遍上面的文档
