---
title: laravel结合webuploader(单页面多上传插件)
categories: [后端,php,Laravel]
tags: [laravel,php]
date: 2019-10-16 17:07:50
---

#### 1. 引入css资源


      <meta name="_token" content="{{ csrf_token() }}"/>
    
        <script src="{{asset('js/jquery.min.js')}}"></script>
      
        <!--引入CSS-->
        <link rel="stylesheet" type="text/css" href="{{ asset('resources/webuploader-0.1.5/webuploader.css') }}">
        <!--引入JS-->
        <script type="text/javascript"
                src="{{ asset('resources/webuploader-0.1.5/webuploader.js')  }}"></script>



#### 2. 定义html

    
      <form id="register-user" class="form-horizontal">
          
          ...
            <div class="form-group">
                <label for="inputPhone" class="col-sm-4 control-label">用户头像</label>
                <div class="col-sm-4">
                    <!--用来存放item-->
                    <input type="hidden" name="photo" id="photo">
                    <div id="photoPreId" class="uploader-list"></div>
                    <div id="photoBtnId">选择图片</div>
                </div>
            </div>
            <div class="form-group">
                <label for="inputPhone2" class="col-sm-4 control-label">资质证明</label>
                <div class="col-sm-4">
                    <!--用来存放item-->
                    <input type="hidden" name="certificate" id="certificate">
                    <div id="certificatePreId" class="uploader-list"></div>
                    <div id="certificateBtnId">选择图片</div>
                </div>
            </div>
           ...
        </form>


#### 3. 编写js


    <script src="{{asset('js/webupload.js')}}"></script>
    
    <script>
        var SCOPO = {
            'upload_url': "{{ route('user.imageUploader') }}",
            'swf_url': "{{ asset('resources/webuploader-0.1.5/Uploader.swf') }}",
        };
        var photo = new $WebUpload("photo", SCOPO.upload_url, SCOPO.swf_url, 100, 100);
        photo.init();
        var certificate = new $WebUpload("certificate", SCOPO.upload_url, SCOPO.swf_url, 100, 100);
        certificate.init();
    </script>



    /**
    
     web-upload 工具类（只适用于上传一个图片）
    
     约定：
    
     上传按钮的id = 图片隐藏域id + 'BtnId'
    
     图片预览框的id = 图片隐藏域id + 'PreId'
    
     author：fengshuonan
     */
    (function () {
    
        var $WebUpload = function (pictureId, uploadUrl, swfUrl, picWidth, picHeight) {
            this.pictureId = pictureId;
            this.uploadBtnId = pictureId + "BtnId";
            this.uploadPreId = pictureId + "PreId";
            this.uploadUrl = uploadUrl;
            this.fileSizeLimit = 10 * 1024 * 1024;
            this.picWidth = picWidth;
            this.picHeight = picHeight;
            this.swfUrl = swfUrl;
        };
    
        $WebUpload.prototype = {
            /**
             * 初始化webUploader
             */
            init: function () {
                var uploader = this.create();
                this.bindEvent(uploader);
                return uploader;
            },
    
            /**
             * 创建webuploader对象
             */
            create: function () {
                var webUploader = WebUploader.create({
                    auto: true,
                    pick: {
                        id: '#' + this.uploadBtnId,
                        multiple: false,// 只上传一个
                    },
                    formData: {
                        // 这里的token是外部生成的长期有效的，如果把token写死，是可以上传的。
                        '_token': $('meta[name="_token"]').attr('content'),
                    },
                    accept: {
                        title: 'Images',
                        extensions: 'gif,jpg,jpeg,bmp,png',
                        mimeTypes: 'image/*'
                    },
                    swf: this.swfUrl,
                    disableGlobalDnd: true,
                    duplicate: true,
                    server: this.uploadUrl,
                    fileSingleSizeLimit: this.fileSizeLimit
                });
    
                return webUploader;
            },
    
            /**
             * 绑定事件
             */
            bindEvent: function (bindedObj) {
                var me = this;
                bindedObj.on('fileQueued', function (file) {
                    var $li = $('<div><img width="100px" height="100px"></div>');
                    var $img = $li.find('img');
    
                    $("#" + me.uploadPreId).html($li);
    
                    // 生成缩略图
                    bindedObj.makeThumb(file, function (error, src) {
                        if (error) {
                            $img.replaceWith('<span>不能预览</span>');
                            return;
                        }
                        $img.attr('src', src);
                    }, me.picWidth, me.picHeight);
                });
    
                // 文件上传过程中创建进度条实时显示。
                bindedObj.on('uploadProgress', function (file, percentage) {
                    var $li = $('#' + file.id),
                        $percent = $li.find('.progress span');
    
                    // 避免重复创建
                    if (!$percent.length) {
                        $percent = $('<p class="progress"><span></span></p>')
                            .appendTo($li)
                            .find('span');
                    }
    
                    $percent.css('width', percentage * 100 + '%');
                });
    
                // 文件上传成功，给item添加成功class, 用样式标记上传成功。
                bindedObj.on('uploadSuccess', function (file, response) {
                    var imgurl = response.data.path;
                    $("#"+this.pictureId).val(imgurl);
                    $("#" + me.pictureId).val(imgurl).addClass('upload-state-done');
                });
    
                // 文件上传失败，显示上传出错。
                bindedObj.on('uploadError', function (file) {
                    var fileerror = response.message;
                    var $li = $('#' + file.id),
                        $error = $li.find('div.error');
    
                // 避免重复创建
                    if (!$error.length) {
                        $error = $('<div class="error"></div>').appendTo($li);
                    }
    
                    $error.text('上传失败' + fileerror);
                });
    
                // 完成上传完了，成功或者失败
                bindedObj.on('uploadComplete', function (file) {
                });
            }
        };
    
        window.$WebUpload = $WebUpload;
    }());


#### 4. `laravel` 端上传代码


    <?php
    /**
     * Created by PhpStorm.
     * User: lenovo
     * Date: 2019/6/28
     * Time: 8:43
     */
    
    namespace App\Http\Controllers\Traits;
    
    use Illuminate\Http\Request;
    
    trait ImageUpload
    {
        /**
         * 抽取的文件上传方法
         * @param Request $request
         * @param $filename 待上传的文件名
         * @param array $exts 文件的后缀数组
         * @return array
         */
        private function uploadFire(Request $request, $filename, $exts = ['jpeg', 'jpg', 'png'])
        {
            $file = $request->file($filename);
            $uploadFilesize = $_FILES[$filename]['size'];
            // 检验一下上传的文件是否有效.
            if ($file && $file->isValid()) {
                //判断上传文件的大小
                if ($uploadFilesize > 1024 * 10 * 1000) {
                    return responseServer(false, "图片大小不能超过10M");
                }
    
                // 上传文件的后缀.
                $entension = $file->getClientOriginalExtension();
    
                if (!in_array($entension, $exts)) {
                    return responseServer(false, "文件格式不正确（必须为.jpg/.jpeg/.png文件）");
                }
    
                //定义生成目录
                $dir = $_SERVER['DOCUMENT_ROOT'] . '/data' . date('/Y/m/d/');
                //文件重新命名
                $filename = rand(100000, 999999) . str_replace('.', '', uniqid("", TRUE)) . "." . $entension;
                //如果目录不存在则创建目录
                makeDir($dir);
    
                $file->move($dir, $filename);
                $picPath = Config("app.upload_url") . '/data' . date('/Y/m/d/') . $filename;
                return responseServer(true, "上传成功", ['path' => $picPath]);
            } else {
                return responseServer(false, "文件不合法");
            }
        }
    }
    
    
    
     //实现图片ajax异步上传
        public function imageUploader(Request $request)
        {
            return $this->uploadFire($request, "file");
        }

