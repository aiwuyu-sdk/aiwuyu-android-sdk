# aiwuyu-android-sdk
## 一. SDK介绍
爱物语Web SDK是针对爱物语h5页面交互开发的一套工具，提供webview功能、渠道用户联合登录到爱物语平台、通过渠道app分享等功能。

## 二. 接入指南
* 获取channelCode    
    爱物语平台会为每个接入渠道提供一个channelCode，用于标识渠道身份。请联系爱物语平台获取。
* 集成方式    
    android sdk已经发不到jcenter上，集成只需要在相应的module中的build.gradle引入即可：
    ```groovy
    implementation 'com.aiwuyu.awylibrary:awyweb:+'
    ```
* 混淆    
    如果您的项目使用了proguard混淆机制，请在混淆中keep爱物语sdk相关代码：
    ```
    -keep class com.aiwuyu.awylibrary.** { *; }
    ```
* 初始化SDK    
    在使用SDK功能前需要初始化SDK，方法如下(demo)：
    ```java
    AwySDK.initialize(this, "awy-app", new AwyDelegate() {

            @Override
            public boolean isAppAuth() {
                return isUserLogin();  //返回App是否登录
            }

            @Override
            public void requestAuth(Context activity) {
                //请求用户登录，请调用起App登录界面
                //FFUserAuthManager.getInstance().startAuth(activity);
            }

            @Override
            public void requestUnionAuthInfo(final AuthCallback authCallback) {
                //需要接入方实现后台接口，获取联合登录加密信息，并通过authCallback调用将联合登录信息返回给SDK，如果获取失败，返回null即可
                //下面的接口调用是demo，请接入方自己实现接口向接入方后台获取联合登录加密信息
                FFSDKAuthProto proto = new FFSDKAuthProto(new IRequestListener<String>() {
                    @Override
                    public void onSuccess(String code, String message, String response, boolean isCache) {
                        try {
                            JSONObject jsonObject = new JSONObject(response);
                            String authInfo = jsonObject.optString("channelReq", "");
                            if (!TextUtils.isEmpty(authInfo)) {
                                //请求成功，返回SDK加密信息
                                authCallback.onObtainAuthInfo(authInfo);
                                return;
                            }

                        } catch (JSONException e) {
                            e.printStackTrace();
                        }
                        //如果获取失败，返回null
                        authCallback.onObtainAuthInfo(null);
                    }

                    @Override
                    public boolean onFailure(String errorCode, String errorMsg, boolean cancel) {
                        //如果获取失败，返回null
                        authCallback.onObtainAuthInfo(null);
                        return false;
                    }
                });
                proto.send();
            }

            @Override
            public void requestShare(Context context, ShareData shareData) {
                //请求分享，接入方app调用分享功能，比如弹出分享框,让用户选择分享
                //下面是demo示例代码，需要接入方自己实现分享调用
                FFShareTool shareTool = new FFShareTool(context, shareData);
                shareTool.showDialog();
            }
        });
    ```
* SDK功能配置    
    SDK的一些功能可以通过配置来开启或关闭，方法如下(demo)：
    ```java
    AwySDK.setConfig(new AwySDKConfig() {
            @Override
            public boolean logEnable() {
                //是否打印log，正式生产环境请不要打印log
                return true;
            }

            @Override
            public boolean isEnvironmentDebug() {
                //SDK提供了两套环境，生产环境和测试环境，集成测试可以在测试环境中完成，正式发布需用生产环境
                //返回true为测试环境，false为生产环境
                return false;
            }

            @Override
            public boolean enableContentShare() {
                //是否支持调用接入方App分享功能
                return true;
            }

            @Override
            public boolean enableUnionAuth() {
                //是否支持通过接入方App实现联合登录，如果不支持，集成的爱物语H5将启用自己的登录
                return true;
            }

            @Override
            public boolean enableCallAuthInterface() {
                //是否支持调用接入方App的登录，如果不支持，在接入方App未登录的情况下，集成的爱物语H5将启用自己的登录
                return true;
            }
        });

    ```
* 打开爱物语H5    
    使用SDK的方法打开爱物语H5页面
    ```java
    AwySDK.openUrl(this, webUrl);
    ```

## 三.  API
#### 1. 入口类 AwySDK

###### 1.1 初始化方法
```java
initialize(Application app, String channelCode, AwyDelegate delegate)
```
用于sdk的初始化，在调用sdk其他功能之前需要对sdk进行初始化，否则会报错，推荐在application的onCreate方法中进行初始化

参数		|描述											|备注
:--		|:--											|:--
app	|Application对象	|无
channelCode	|渠道身份标识		|无
delegate	|用于SDK与渠道APP交互的接口实例		|详细定义见下文AwyDelegate定义


###### 1.2 使用sdk打开h5链接

```java
openUrl(Activity activity, String url)
```
用于使用sdk中webview打开一个h5链接，注意：一定在initialize初始化之后才能调用此方法

参数		|描述											|备注
:--		|:--											|:--
activity	|启动h5页面的context	|无
url	|需要打开的h5 url		|无

###### 1.3 获取bitmap对象

```java
getBitmapFromUrl(String imageUrl, final BitmapListener listener)
```
用于根据图片url 加载成bitmap并通过listener返回

场景：如果您的分享图标需要bitmap类型，可以使用此方法获得

参数		|描述											|备注
:--		|:--											|:--
imageUrl	|需要转化成bitmap的图片链接	|无
listener	|当bitmap load成功后的回掉		|详细定义见下文




#### 2. SDK回掉渠道app接口 AwyDelegate

###### 2.1 请求联合登录参数

```java
boolean requestAuth(AuthCallback callback);
```
sdk向渠道app请求联合登录参数

渠道方需要使用爱物语server sdk生成联合登录参数，

并通过callback异步传递给爱物语sdk

参数		|描述											|备注
:--		|:--											|:--
callback	|渠道app获取到联合登录参数后，使用此回调将联合登录参数传递给sdk	|详细定义见下文
返回值	|true表示本渠道支持联合登录，false表示不支持联合登录	|无


###### 2.2 请求分享

```java
boolean requestShare(ShareData shareData);
```
sdk向渠道app请求分享	

参数		|描述											|备注
:--		|:--											|:--
shareData	|分享参数，包括分享标题、内容、链接、图片等信息	|详细定义见下文
返回值	|true表示支持通过渠道app分享内容，false表示不支持分享内容	|无





#### 3. 获取到联合登录参数后回调sdk接口  AuthCallback

###### 3.1 联合登录参数回调

```java
void onObtainAuthInfo(String authInfo);
```
渠道app获取到联合登录参数后

通过该方法回调参数给sdk

参数		|描述											|备注
:--		|:--											|:--
authInfo	|渠道方通过爱物语server sdk获得的联合登录参数，	|如果未登录，返回null即可



#### 4. 加载图片bitmap回调接口  BitmapListener

###### 4.1 加载成功回调

```java
void onResult(Bitmap bitmap);
```
sdk加载到图片bitmap使用此方法回调

参数		|描述											|备注
:--		|:--											|:--
bitmap	|加载成功的bitmap对象	|无

###### 4.2 加载失败回调

```java
void onError();
```
sdk加载图片bitmap失败时回调



#### 5. 分享数据模型  ShareData

参数		|描述											|备注
:--		|:--											|:--
title	|分享标题	|无
content	|分享内容	|无
shareUrl	|分享的链接url	|无
imageUrl	|分享图标url	|无

上述参数均可以使用对应的getter/setter方法操作



## 四. 联合登录整体流程

* 爱物语sdk向渠道app请求联合登录参数

* 如果渠道app是已经登录状态，渠道app向渠道后台请求联合登录参数，如果未登录直接回调sdk null即可

* 渠道后台收到联合登录请求后，调用爱物语server sdk生成联合登录参数，并下发给渠道app

* 渠道app再将参数通过接口回调，传递给sdk

* sdk使用联合登录参数请求爱物语平台登录

* 登录完成
