# okhttp3

###How to

To get a Git project into your build:

Step 1. Add the JitPack repository to your build file

gradle
maven
sbt
leiningen
Add it in your root build.gradle at the end of repositories:

	allprojects {
		repositories {
			...
			maven { url 'https://jitpack.io' }
		}
	}Copy
Step 2. Add the dependency

	dependencies {
	        compile 'com.github.Dkaishu:okhttp3:v1.0.0'
	}

###感谢：
代码源自 https://github.com/MrZhousf/OkHttp3 ，做了部分改动。

###说明：

    使用时apk大小增加在342k左右；
    最低支持版本14；

###使用指南：

####权限

    <!-- 添加读写权限 -->
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
    <!-- 访问互联网权限 -->
    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.CHANGE_NETWORK_STATE"/>
    <uses-permission android:name="android.permission.CHANGE_WIFI_STATE"/>
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE"/>
####自定义全局配置

在Application中配置如下：

	String downloadFileDir = Environment.getExternalStorageDirectory().getPath()+"/okHttp_download/";
        String cacheDir = Environment.getExternalStorageDirectory().getPath()+"/okHttp_cache";
        OkHttpUtil.init(context)
                .setConnectTimeout(15)//连接超时时间
                .setWriteTimeout(15)//写超时时间
                .setReadTimeout(15)//读超时时间
                .setMaxCacheSize(10 * 1024 * 1024)//缓存空间大小
                .setCacheType(CacheType.FORCE_NETWORK)//缓存类型
                .setHttpLogTAG("HttpLog")//设置请求日志标识
                .setIsGzip(false)//Gzip压缩，需要服务端支持
                .setShowHttpLog(true)//显示请求日志
                .setShowLifecycleLog(false)//显示Activity销毁日志
                .setRetryOnConnectionFailure(false)//失败后不自动重连
                .setCachedDir(new File(cacheDir))//设置缓存目录
                .setDownloadFileDir(downloadFileDir)//文件下载保存目录
                .setResponseEncoding(Encoding.UTF_8)//设置全局的服务器响应编码
                .setRequestEncoding(Encoding.UTF_8)//设置全局的请求参数编码
                .addResultInterceptor(HttpInterceptor.ResultInterceptor)//请求结果拦截器
                .addExceptionInterceptor(HttpInterceptor.ExceptionInterceptor)//请求链路异常拦截器
                .setCookieJar(new PersistentCookieJar(new SetCookieCache(), new SharedPrefsCookiePersistor(context)))//持久化cookie
                .build();
获取网络请求客户端单例示例

//获取单例客户端（默认）

        方法一、OkHttpUtil.getDefault(this)//绑定生命周期
            .doGetSync(HttpInfo.Builder().setUrl(url).build());
	    
        方法二、OkHttpUtil.getDefault()//不绑定生命周期
            .doGetSync(HttpInfo.Builder().setUrl(url).build());
取消指定请求

        建议在视图中采用OkHttpUtil.getDefault(this)的方式进行请求绑定，该方式会在Activity/Fragment销毁时自动取消当前视图下的所有请求；
        请求标识类型支持Object、String、Integer、Float、Double；
        请求标识务必保证唯一。
        
    //*******请求时先绑定请求标识，根据该标识进行取消*******/
        
        //方法一：
        OkHttpUtil.Builder()
                .setReadTimeout(120)
                .build("请求标识")//绑定请求标识
                .doDownloadFileAsync(info);
                
        //方法二：
        OkHttpUtil.getDefault("请求标识")//绑定请求标识
            .doGetSync(info);
        
    //*******取消指定请求*******/
	    
        OkHttpUtil.getDefault().cancelRequest("请求标识");
HttpInfo参数解析：

    键值对提交采用addParam/addParams方法
    Json提交采用addParamJson方法
   
    表单提交采用addParamForm方法
   
    二进制字节流提交采用addParamBytes方法
   
    文件上传采用addDownloadFile方法
   
    文件下载采用addUploadFile方法
    
    HttpInfo.Builder()
        .setUrl(url)
        .setRequestType(RequestType.GET)//请求方式
        .addHead("head","test")//添加头参数
        .addParam("param","test")//添加接口键值对参数
        .addParams(new HashMap<String, String>())//添加接口键值对参数集合
        .addParamBytes("byte")//添加二进制流
        .addParamJson("json")//添加Json参数
        .addParamFile(new File(""))//添加文档参数
        .addParamForm("form")//添加表单参数
        .addDownloadFile(new DownloadFileInfo("fileURL", "myMP4",null))//添加下载文件
        .addUploadFile("interfaceParamName","filePathWithName",null)//添加上传文件
        .setResponseEncoding(Encoding.UTF_8)//设置服务器响应编码
        .setRequestEncoding(Encoding.UTF_8)//设置全局的请求参数编码
        .setDelayExec(2, TimeUnit.SECONDS)//延迟2秒执行
        .build()
	
在Activity中同步调用示例

    /**
     * 同步请求：由于不能在UI线程中进行网络请求操作，所以采用子线程方式
     */
    private void sync() {
        new Thread(new Runnable() {
            @Override
            public void run() {
                final HttpInfo info = HttpInfo.Builder()
                        .setUrl(url)
                        .setResponseEncoding(Encoding.UTF_8)//设置该接口的服务器响应编码
                        .setRequestEncoding(Encoding.UTF_8)//设置该接口的请求参数编码
                        .build();
                OkHttpUtil.getDefault(this)
                        .doGetSync(info);
                final String result = info.getRetDetail();
                runOnUiThread(new Runnable() {
                    @Override
                    public void run() {
                        resultTV.setText("同步请求：" + result);
                        setFromCacheTV(info);

                    }
                });
            }
        }).start();
    }
在Activity中异步调用示例

    /**
     * 异步请求：回调方法可以直接操作UI
     */
    private void async() {
        OkHttpUtil.getDefault(this).doGetAsync(
                HttpInfo.Builder().setUrl(url).build(),
                new Callback() {
                    @Override
                    public void onFailure(HttpInfo info) throws IOException {
                        String result = info.getRetDetail();
                        resultTV.setText("异步请求失败：" + result);
                    }

                    @Override
                    public void onSuccess(HttpInfo info) throws IOException {
                        String result = info.getRetDetail();
                        resultTV.setText("异步请求成功：" + result);
                        //GSon解析
                        TimeAndDate time = info.getRetDetail(TimeAndDate.class);
                        LogUtil.d("MainActivity", time.getResult().toString());
                        setFromCacheTV(info);
                    }
                });
    }
仅网络请求

    /**
     * 仅网络请求
     */
    private void forceNetwork(){
        OkHttpUtil.Builder().setCacheType(CacheType.FORCE_NETWORK).build(this)
                .doGetAsync(
                        HttpInfo.Builder().setUrl(url).build(),
                        new Callback() {
                            @Override
                            public void onSuccess(HttpInfo info) throws IOException {
                                String result = info.getRetDetail();
                                resultTV.setText("FORCE_NETWORK：" + result);
                                setFromCacheTV(info);
                            }

                            @Override
                            public void onFailure(HttpInfo info) throws IOException {
                                resultTV.setText("FORCE_NETWORK：" + info.getRetDetail());
                            }
                        }
                );
    }
仅缓存请求

    /**
     * 仅缓存请求
     */
    private void forceCache(){
        OkHttpUtil.Builder().setCacheType(CacheType.FORCE_CACHE).build(this)
                .doGetAsync(
                        HttpInfo.Builder().setUrl(url).build(),
                        new Callback() {
                            @Override
                            public void onSuccess(HttpInfo info) throws IOException {
                                String result = info.getRetDetail();
                                resultTV.setText("FORCE_CACHE：" + result);
                                setFromCacheTV(info);
                            }

                            @Override
                            public void onFailure(HttpInfo info) throws IOException {
                                resultTV.setText("FORCE_CACHE：" + info.getRetDetail());
                            }
                        }
                );
    }
先网络再缓存

    /**
     * 先网络再缓存：先请求网络，失败则请求缓存
     */
    private void networkThenCache() {
        OkHttpUtil.Builder().setCacheType(CacheType.NETWORK_THEN_CACHE).build(this)
                .doGetAsync(
                        HttpInfo.Builder().setUrl(url).build(),
                        new Callback() {
                            @Override
                            public void onSuccess(HttpInfo info) throws IOException {
                                String result = info.getRetDetail();
                                resultTV.setText("NETWORK_THEN_CACHE：" + result);
                                setFromCacheTV(info);
                            }

                            @Override
                            public void onFailure(HttpInfo info) throws IOException {
                                resultTV.setText("NETWORK_THEN_CACHE：" + info.getRetDetail());
                            }
                        }
                );
    }
先缓存再网络

    /**
     * 先缓存再网络：先请求缓存，失败则请求网络
     */
    private void cacheThenNetwork() {
        OkHttpUtil.Builder().setCacheType(CacheType.CACHE_THEN_NETWORK).build(this)
                .doGetAsync(
                        HttpInfo.Builder().setUrl(url).build(),
                        new Callback() {
                            @Override
                            public void onSuccess(HttpInfo info) throws IOException {
                                String result = info.getRetDetail();
                                resultTV.setText("CACHE_THEN_NETWORK：" + result);
                                setFromCacheTV(info);
                            }

                            @Override
                            public void onFailure(HttpInfo info) throws IOException {
                                resultTV.setText("CACHE_THEN_NETWORK：" + info.getRetDetail());
                            }
                        }
                );
    }
缓存10秒失效

    /**
     * 缓存10秒失效：连续点击进行测试10秒内再次请求为缓存响应，10秒后再请求则缓存失效并进行网络请求
     */
    private void tenSecondCache(){
        //由于采用同一个url测试，需要先清理缓存
        if(isNeedDeleteCache){
            isNeedDeleteCache = false;
            OkHttpUtil.getDefault().deleteCache();
        }
        OkHttpUtil.Builder()
                .setCacheType(CacheType.CACHE_THEN_NETWORK)
                .setCacheSurvivalTime(10)//缓存存活时间为10秒
                .build(this)
                .doGetAsync(
                        HttpInfo.Builder().setUrl(url).build(),
                        new Callback() {
                            @Override
                            public void onSuccess(HttpInfo info) throws IOException {
                                String result = info.getRetDetail();
                                resultTV.setText("缓存10秒失效：" + result);
                                setFromCacheTV(info);
                            }

                            @Override
                            public void onFailure(HttpInfo info) throws IOException {
                                resultTV.setText("缓存10秒失效：" + info.getRetDetail());
                            }
                        }
                );
    }
在Activity中上传图片示例

     /**
         * 异步上传图片：显示上传进度
         */
        private void doUploadImg() {
            HttpInfo info = HttpInfo.Builder()
                            .setUrl(url)
                            .addUploadFile("file", filePathOne, new ProgressCallback() {
                                //onProgressMain为UI线程回调，可以直接操作UI
                                @Override
                                public void onProgressMain(int percent, long bytesWritten, long contentLength, boolean done) {
                                    uploadProgressOne.setProgress(percent);
                                    LogUtil.d(TAG, "上传进度：" + percent);
                                }
                            })
                            .build();
            OkHttpUtil.getDefault(this).doUploadFileAsync(info);
        }
在Activity中单次批量上传文件示例

    若服务器为php，接口文件参数名称后面追加"[]"表示数组， 示例：builder.addUploadFile("uploadFile[]",path);
        /**
         * 单次批量上传：一次请求上传多个文件
         */
         private void doUploadBatch(){
            imgList.clear();
            imgList.add("/storage/emulated/0/okHttp_download/test.apk");
            imgList.add("/storage/emulated/0/okHttp_download/test.rar");
            HttpInfo.Builder builder = HttpInfo.Builder()
                    .setUrl(url);
            //循环添加上传文件
            for (String path: imgList  ) {
                //若服务器为php，接口文件参数名称后面追加"[]"表示数组，示例：builder.addUploadFile("uploadFile[]",path);
                builder.addUploadFile("uploadFile",path);
            }
            HttpInfo info = builder.build();
            OkHttpUtil.getDefault(UploadFileActivity.this).doUploadFileAsync(info,new ProgressCallback(){
                @Override
                public void onProgressMain(int percent, long bytesWritten, long contentLength, boolean done) {
                    uploadProgress.setProgress(percent);
                }

            @Override
            public void onResponseMain(String filePath, HttpInfo info) {
                LogUtil.d(TAG, "上传结果：" + info.getRetDetail());
                tvResult.setText(info.getRetDetail());
            }
        });
    }
在Activity中断点下载文件示例

     @OnClick({R.id.downloadBtn, R.id.pauseBtn, R.id.continueBtn})
        public void onClick(View view) {
            switch (view.getId()) {
                case R.id.downloadBtn://下载
                    download();
                    break;
                case R.id.pauseBtn://暂停下载
                    if(null != fileInfo)
                        fileInfo.setDownloadStatus(DownloadStatus.PAUSE);
                    break;
                case R.id.continueBtn://继续下载
                    download();
                    break;
            }
        }

    private void download(){
        if(null == fileInfo)
            fileInfo = new DownloadFileInfo(url,"fileName",new ProgressCallback(){
                @Override
                public void onProgressMain(int percent, long bytesWritten, long contentLength, boolean done) {
                    downloadProgress.setProgress(percent);
                    tvResult.setText(percent+"%");
                    LogUtil.d(TAG, "下载进度：" + percent);
                }
                @Override
                public void onResponseMain(String filePath, HttpInfo info) {
                    if(info.isSuccessful()){
                        tvResult.setText(info.getRetDetail()+"\n下载状态："+fileInfo.getDownloadStatus());
                    }else{
                        Toast.makeText(DownloadBreakpointsActivity.this,info.getRetDetail(),Toast.LENGTH_SHORT).show();
                    }
                }
            });
        HttpInfo info = HttpInfo.Builder().addDownloadFile(fileInfo).build();
        OkHttpUtil.Builder().setReadTimeout(120).build(this).doDownloadFileAsync(info);
    }
二进制流方式请求

    HttpInfo info = new HttpInfo.Builder()
            .setUrl("http://192.168.120.154:8082/StanClaimProd-app/surveySubmit/getFileLen")
            .addParamBytes(byte)//添加二进制流
            .build();
    OkHttpUtil.getDefault().doPostAsync(info, new Callback() {
        @Override
        public void onSuccess(HttpInfo info) throws IOException {
            String result = info.getRetDetail();
            resultTV.setText("请求失败："+result);
        }
    
        @Override
        public void onFailure(HttpInfo info) throws IOException {
            resultTV.setText("请求成功："+info.getRetDetail());
        }
    });
请求结果统一预处理拦截器/请求链路异常信息拦截器示例

    请求结果拦截器与链路异常拦截器方便项目进行网络请求业务时对信息返回的统一管理与设置

    /**
     * Http拦截器
     * 1、请求结果统一预处理拦截器
     * 2、请求链路异常信息拦截器
     * @author zhousf
     */
    public class HttpInterceptor {
    
        /**
         * 请求结果统一预处理拦截器
         * 该拦截器会对所有网络请求返回结果进行预处理并修改
         */
        public static ResultInterceptor ResultInterceptor = new ResultInterceptor() {
            @Override
            public HttpInfo intercept(HttpInfo info) throws Exception {
                //请求结果预处理：可以进行GSon过滤与解析
                return info;
            }
        };
    
        /**
         * 请求链路异常信息拦截器
         * 该拦截器会发送网络请求时链路异常信息进行拦截处理
         */
        public static ExceptionInterceptor ExceptionInterceptor = new ExceptionInterceptor() {
            @Override
            public HttpInfo intercept(HttpInfo info) throws Exception {
                switch (info.getRetCode()){
                    case HttpInfo.NonNetwork:
                        info.setRetDetail("网络中断");
                        break;
                    case HttpInfo.CheckURL:
                        info.setRetDetail("网络地址错误["+info.getNetCode()+"]");
                        break;
                    case HttpInfo.ProtocolException:
                        info.setRetDetail("协议类型错误["+info.getNetCode()+"]");
                        break;
                    case HttpInfo.CheckNet:
                        info.setRetDetail("请检查网络连接是否正常["+info.getNetCode()+"]");
                        break;
                    case HttpInfo.ConnectionTimeOut:
                        info.setRetDetail("连接超时");
                        break;
                    case HttpInfo.WriteAndReadTimeOut:
                        info.setRetDetail("读写超时");
                        break;
                    case HttpInfo.ConnectionInterruption:
                        info.setRetDetail("连接中断");
                        break;
                }
                return info;
            }
        };
    }
Cookie持久化示例

    没有在Application中进行全局Cookie持久化配置时可以采用以下方式：
    
    OkHttpUtilInterface okHttpUtil = OkHttpUtil.Builder()
                .setCacheLevel(FIRST_LEVEL)
                .setConnectTimeout(25).build(this);
    //一个okHttpUtil即为一个网络连接
    okHttpUtil.doGetAsync(
                    HttpInfo.Builder().setUrl(url).build(),
                    new CallbackOk() {
                        @Override
                        public void onResponse(HttpInfo info) throws IOException {
                            if (info.isSuccessful()) {
                                String result = info.getRetDetail();
                                resultTV.setText("异步请求："+result);
                            }
                        }
                    });



