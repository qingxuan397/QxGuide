## 安排上了！PC人脸识别登录，出乎意料的简单

原创 程序员内点事 [程序员内点事](javascript:void(0);) *2020-07-29*

之前不是做了个开源项目嘛，在做完`GitHub`登录后，想着再显得有逼格一点，说要再加个人脸识别登录，就我这佛系的开发进度，过了一周总算是抽时间安排上了。

**源码在文末**

其实最近对写文章有点小抵触，写的东西没人看，总有点小失落，好在有同行大佬们的开导让我重拾了信心。调整了自己的心态，只要我分享的东西对大家有帮助就好，至于多少人看那就随缘吧！

废话不多说先看人脸识别效果动态，马赛克有点重哈，没办法长相实在是拿不出手。

![图片](https://mmbiz.qpic.cn/mmbiz_gif/0OzaL5uW2aNjGibYaV8t88sttOH95r5rib6c1ZbHcydeLX01Nyvl6wuGrDfcugs5P20FUBoOXCYYstzJUxjxHYtw/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)

## 实现原理

我们看一下实现人脸识别登录的大致流程，三个主要步骤：

![图片](https://mmbiz.qpic.cn/mmbiz_png/0OzaL5uW2aNjGibYaV8t88sttOH95r5ribCGMZiav7bdXAVzDicMic3p2FkMx3rys1zibBgSa2OrheFbYzKPJMJ6rEtg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

1. 前端登录页打开摄像头，进行人脸识别，**注意**：只识别画面中是不是有人脸
2. 识别到人脸后，拍照上传当前画面图片
3. 后端接受图片并调用人脸库SDK，对人像进行比对，通过则登录成功，并将人像信息注册到人脸库和本地`mysql`。

## 前端实现

上边说过要在前端识别到人脸，所以这里就不得不借助工具了，我使用的 tracking.js，一款轻量级的前端人脸识别框架。

前端 `Vue` 代码实现逻辑比较简单，`tracking.js` 打开摄像头识别到人脸信息后，对视频图像拍照，将图片信息上传到后台，等待图片对比的结果就可以了。

```
data() {
        return {
            showContainer: true,   // 显示
            tracker: null,
            tipFlag: false,         // 提示用户已经检测到
            flag: false,            // 判断是否已经拍照
            context: null,          // canvas上下文
            removePhotoID: null,    // 停止转换图片
            scanTip: '人脸识别中...',// 提示文字
            imgUrl: '',              // base64格式图片
            canvas: null
        }
    },
    mounted() {
        this.playVideo()
    },
    methods: {

        playVideo() {
            var video = document.getElementById('video');
            this.canvas = document.getElementById('canvas');
            this.context = this.canvas.getContext('2d');
            this.tracker = new tracking.ObjectTracker('face');
            this.tracker.setInitialScale(4);
            this.tracker.setStepSize(2);
            this.tracker.setEdgesDensity(0.1);

            tracking.track('#video', this.tracker, {camera: true});

            this.tracker.on('track', this.handleTracked);
        },

        handleTracked(event) {
                this.context.clearRect(0, 0, this.canvas.width, this.canvas.height);
                if (event.data.length === 0) {
                    this.scanTip = '未识别到人脸'
                } else {
                    if (!this.tipFlag) {
                        this.scanTip = '识别成功，正在拍照，请勿乱动~'
                    }
                    // 1秒后拍照，仅拍一次
                    if (!this.flag) {
                        this.scanTip = '拍照中...'
                        this.flag = true
                        this.removePhotoID = setTimeout(() => {
                                this.tackPhoto()
                                this.tipFlag = true
                            },
                            2000
                        )
                    }
                    event.data.forEach(this.plot);
                }
        },

        plot(rect){
            this.context.strokeStyle = '#eb652e';
            this.context.strokeRect(rect.x, rect.y, rect.width, rect.height);
            this.context.font = '11px Helvetica';
            this.context.fillStyle = "#fff";
            this.context.fillText('x: ' + rect.x + 'px', rect.x + rect.width + 5, rect.y + 11);
            this.context.fillText('y: ' + rect.y + 'px', rect.x + rect.width + 5, rect.y + 22);
        },

        // 拍照
        tackPhoto() {

            this.context.drawImage(this.$refs.refVideo, 0, 0, 500, 500)
            // 保存为base64格式
            this.imgUrl = this.saveAsPNG(this.$refs.refCanvas)
            var formData = new FormData();
            formData.append("file", this.imgUrl);
            this.scanTip = '登录中，请稍等~'

            axios({
                method: 'post',
                url: '/faceDiscern',
                data: formData,
            }).then(function (response) {
                alert(response.data.data);
                window.location.href="http://127.0.0.1:8081/home";
            }).catch(function (error) {
                console.log(error);
            });

            this.close()
        },

        // 保存为png,base64格式图片
        saveAsPNG(c) {
            return c.toDataURL('image/png', 0.3)
        },

        // 关闭并清理资源
        close() {
            this.flag = false
            this.tipFlag = false
            this.showContainer = false
            this.tracker && this.tracker.removeListener('track', this.handleTracked) && tracking.track('#video', this.tracker, {camera: false});
            this.tracker = null
            this.context = null
            this.scanTip = ''
            clearTimeout(this.removePhotoID)
        }
    }
```

## 人脸识别

之前也搞过一个人脸识别案例 [《基于 Java 实现的人脸识别功能（附源码）》](https://mp.weixin.qq.com/s?__biz=MzAxNTM4NzAyNg==&mid=2247483925&idx=1&sn=c83bf51f3c67ce7ea65c3ac7b200a852&scene=21#wechat_redirect) ，不过调用SDK的方式太过繁琐，而且代码量巨大。所以这次为了简化实现，改用了百度的人脸识别API，没想到出乎意料的简单。

> “
>
> 别抬杠问我为啥不自己写人脸识别工具，别问，问就是不会

在百度云注册一个应用 `https://console.bce.baidu.com/ai/?_=1595996996657&fromai=1#/ai/face/app/list`，得到 `API Key`和 `Secret Key`，为了后续获取 `token`用。

![图片](https://mmbiz.qpic.cn/mmbiz_png/0OzaL5uW2aNjGibYaV8t88sttOH95r5ribJNsHXR3CpLDWzXjsuTVyJYwffkokw8GoIvYP4ibaDKZu4s0kcCwzPMw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

百度云人脸识别的API非常友好，各种操作的 demo都写好了，拿过来简单改改就可以。

第一步先获取`token`，这是调用百度人脸识别`API`的基础。

```
https://aip.baidubce.com/oauth/2.0/token?
grant_type=client_credentials&
client_id=【百度云应用的AK】&
client_secret=【百度云应用的SK】
```

接下来我们开始对图片进行比对，百度云提供了一个在线的人脸库，用户登录我们先在人脸库查询人像是否存在，存在则表示登录成功，如果不存在则注册到人脸库。每个图片有一个唯一标识`face_token`。

![图片](https://mmbiz.qpic.cn/mmbiz_png/0OzaL5uW2aNjGibYaV8t88sttOH95r5rib6ibYgCK4JUG6qNanKWp3zynGLb2Via5Kdyiby0q4nKLFC25QNWEgWlbXA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

百度人脸识别 `API` 实现比较简单，需要特别注意参数`image_type`，它有三种类型

- `BASE64`：图片的base64值，base64编码后的图片数据，编码后的图片大小不超过2M；
- `URL`：图片的 `URL`地址( 可能由于网络等原因导致下载图片时间过长)；
- `FACE_TOKEN`：人脸图片的唯一标识，调用人脸检测接口时，会为每个人脸图片赋予一个唯一的`FACE_TOKEN`，同一张图片多次检测得到的`FACE_TOKEN`是同一个。

而我们这里使用的是图片`BASE64`文件，所以`image_type`要设置成`BASE64`。

```
    @Override
    public BaiDuFaceSearchResult faceSearch(String file) {

        try {
            byte[] decode = Base64.decode(Base64Util.base64Process(file));
            String faceFile = Base64Util.encode(decode);

            Map<String, Object> map = new HashMap<>();
            map.put("image", faceFile);
            map.put("liveness_control", "NORMAL");
            map.put("group_id_list", "user");
            map.put("image_type", "BASE64");
            map.put("quality_control", "LOW");
            String param = GsonUtils.toJson(map);

            String result = HttpUtil.post(faceSearchUrl, this.getAccessToken(), "application/json", param);
            BaiDuFaceSearchResult searchResult = JSONObject.parseObject(result, BaiDuFaceSearchResult.class);
            log.info(" faceSearch: {}", JSON.toJSONString(searchResult));
            return searchResult;
        } catch (Exception e) {
            log.error("get faceSearch error {}", e.getStackTrace());
            e.getStackTrace();
        }
        return null;
    }

    @Override
    public BaiDuFaceDetectResult faceDetect(String file) {

        try {
            byte[] decode = Base64.decode(Base64Util.base64Process(file));
            String faceFile = Base64Util.encode(decode);

            Map<String, Object> map = new HashMap<>();
            map.put("image", faceFile);
            map.put("face_field", "faceshape,facetype");
            map.put("image_type", "BASE64");
            String param = GsonUtils.toJson(map);

            String result = HttpUtil.post(faceDetectUrl, this.getAccessToken(), "application/json", param);
            BaiDuFaceDetectResult detectResult = JSONObject.parseObject(result, BaiDuFaceDetectResult.class);
            log.info(" detectResult: {}", JSON.toJSONString(detectResult));
            return detectResult;
        } catch (Exception e) {
            log.error("get faceDetect error {}", e.getStackTrace());
            e.getStackTrace();
        }
        return null;
    }

    @Override
    public BaiDuFaceAddResult addFace(String file, UserFaceInfo userFaceInfo) {

        try {
            byte[] decode = Base64.decode(Base64Util.base64Process(file));
            String faceFile = Base64Util.encode(decode);

            Map<String, Object> map = new HashMap<>();
            map.put("image", faceFile);
            map.put("group_id", "user");
            map.put("user_id", userFaceInfo.getUserId());
            map.put("user_info", JSON.toJSONString(userFaceInfo));
            map.put("liveness_control", "NORMAL");
            map.put("image_type", "BASE64");
            map.put("quality_control", "LOW");
            String param = GsonUtils.toJson(map);

            String result = HttpUtil.post(addfaceUrl, this.getAccessToken(), "application/json", param);
            BaiDuFaceAddResult addResult = JSONObject.parseObject(result, BaiDuFaceAddResult.class);
            log.info("addResult: {}", JSON.toJSONString(addResult));
            return addResult;
        } catch (Exception e) {
            log.error("get addFace error {}", e.getStackTrace());
            e.getStackTrace();
        }
        return null;
    }
```

项目是前后端分离的，但为了大家学习方便，我把人脸识别页面整合到了后端项目。

最后 run FireControllerApplication 访问地址：http://localhost:8082/face 即可。

源码`GitHub`地址：`https://github.com/chengxy-nds/fire.git`，欢迎大家来耍~