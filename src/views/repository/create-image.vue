<style lang="less">
    @import '../../styles/common';
    @import './style/image-editor';
</style>
<style>
    .ivu-card-body{
        height: 100% !important;
    }
</style>
<template>
    <div class="image-editor">
        <Modal
            v-model="option.showCropedImage"
            :mask-closable="false">
            <p slot="header">
                <Icon type="logo-instagram"></Icon>
                预览裁剪之后的图片
            </p>
            <img :src="option.cropedImg" alt=""  style="width: 100%;">
            <div slot="footer">
                <Button type="text" @click="option.showCropedImage = false">取消</Button>
                <Button type="primary" @click="uploadImg('trim')">保存上传</Button>
            </div>
        </Modal>
        <Row :gutter="10">
            <Col span="24">
                <Card>
                    <Form inline v-model="imgObj">
                        <FormItem style="margin-bottom: 10px" :label-width='60' label="文件名称"  prop="name">
                            <Input placeholder="输入文件名称" style="width: 200px" v-model="imgObj.name"/>
                        </FormItem>
                        <FormItem style="margin-bottom: 0px">
                            <label class="filelabel" style="padding: 0px 15px;border: 0px" for="fileinput">
                                <input type="file" icon="image" accept="image/png, image/jpeg, image/gif, image/jpg" @change="handleChange" id="fileinput" class="fileinput"/>
                                <Icon type="md-image"></Icon>&nbsp;选择
                            </label>
                            <Button type="primary" @click="uploadImg('direct')" icon="md-cloud-upload">上传</Button>
                            <Button @click="handlecrop" type="primary" icon="md-crop">裁剪</Button>
                        </FormItem>
                    </Form>
                </Card>
                <Row :gutter="10" class="margin-top-10">
                    <Col :sm="17" :xs="24" class="image-editor-con1 cropImgH">
                        <Card :style="{height: cropperH}">
                            <div class="cropper" style="height: 100%">
                                <img id="cropimg" height="505" :src="imgSrc" alt="">
                            </div>
                        </Card>
                    </Col>
                    <Col :sm="7" :xs="24" class="image-editor-con1 scannerH">
                        <Card>
                            <p slot="title">
                                <Icon type="logo-instagram"></Icon>
                                裁剪预览
                            </p>
                            <Row type="flex" justify="center" align="middle" class="image-editor-con1-preview-con">
                                <div id="preview"></div>
                            </Row>
                        </Card>
                    </Col>
                </Row>
            </Col>
        </Row>
    </div>
</template>

<script>
    import Cropper from 'cropperjs';
    import './style/cropper.min.css';
    import $util from '@/libs/util.js';
    import defaultimg from '../../images/cropper-test.png'

    export default {
        name: "create-image",
        data() {
            return {
                imgSrc:defaultimg,
                cropperH:'0px',
                cropper: {},
                uploadFile:{},
                option: {
                    showCropedImage: false,
                    cropedImg: ''
                },
                imgObj:{
                    id:"",
                    name:"",
                    imgPath:"",
                    type:"img"
                }
            };
        },
        methods: {
            initImageH(){
                var docHeight = document.body.scrollHeight;
                var imgH = docHeight - 250;
                if(imgH < 358){
                    imgH = 358;
                }
                this.cropperH = imgH + 'px';
                var imgBack = document.getElementsByClassName("cropper-container cropper-bg")[0];
                if(Object.keys(this.cropper).length != 0){
                    // let containerData = this.cropper.containerData;
                    // this.cropper.setStyle(this.cropper, {
                    //     width: containerData.width,
                    //     height: containerData.height+30,
                    // });
                    // this.cropper.reset();
                    // console.info(this.cropper.containerData.height)
                    // imgBack.style.height = imgH + 'px';
                    // console.info(imgBack.style.height);
                    // imgBack.style.width = "100%";
                }
            },
            init() {
                let img = document.getElementById('cropimg');
                this.cropper = new Cropper(img, {
                    dragMode: 'move',    // 定义cropper的拖拽模式。
                    preview: '#preview', // 添加额外的元素(容器)以供预览。
                    guides:true,         // 显示在裁剪框上方的虚线。
                    autoCropArea:0.5,    // 定义自动裁剪面积大小(百分比)和图片进行对比。
                    restore: true,       // 在调整窗口大小后恢复裁剪的区域。
                    center: true,        // 裁剪框在图片正中心。
                    movable:true,       // 是否允许可以移动后面的图片
                    highlight: false,    // 在裁剪框上方显示白色的区域(突出裁剪框)。
                    cropBoxMovable: true,// 是否通过拖拽来移动剪裁框。
                    zoomOnWheel:false,   // 是否可以通过移动鼠标来放大图像。
                    toggleDragModeOnDblclick: true, // 当点击两次时可以在“crop”和“move”之间切换拖拽模式。
                });

                // 处理图片显示高度
                var heightLeft = document.getElementsByClassName('image-editor-con1 ivu-col cropImgH');
                var heightRight = document.getElementsByClassName('image-editor-con1 ivu-col scannerH');
                heightRight[0].classList.remove("image-editor-con1");
                heightLeft[0].classList.remove("image-editor-con1");
                let url = "initAdvertisement";
                let _this = this;
                $util.get(url)
                    .then(function (response) {
                        if(response.status == 200 ){
                            if( response.data.statusCode == "10000"){
                                // 暂没想到
                            }else{
                                $util.responseMsg(_this,response.data);
                            }
                        }else{
                            $util.httpErrorMsg(_this,response.data)
                        }
                    })
                    .catch(function (error) {
                        $util.httpErrorMsg(_this,response.data)
                    })
            },
            handleChange(e) {
                let file = e.target.files[0];
                this.uploadFile = file;
                let reader = new FileReader();
                reader.onload = () => {
                    this.cropper.replace(reader.result);
                    reader.onload = null;
                };
                reader.readAsDataURL(file);
            },
            uploadImg(data){
                if(this.imgObj.name == ""){
                    $util.frontErrMsg(this,2,"请输入名称");
                    return;
                }
                let upFile = {};
                if(data ==  'direct'){ // 直接上传
                    upFile = this.uploadFile;
                }else if(data ==  'trim'){ // 裁剪上传
                    upFile = $util.base64toFile(this.option.cropedImg,this.imgObj.name);
                }else{
                    $util.frontErrMsg(this,2,"请上传图片");
                    return;
                }
                if(upFile == null){
                    $util.frontErrMsg(this,2,"请上传图片");
                    return;
                }
                let url = "upload";
                let _this = this;
                let param = new window.FormData();
                param.append('file', upFile);
                $util.post(url,param,{
                        headers: {
                            'Content-Type': 'multipart/form-data'
                        }
                    })
                    .then(function (response) {
                        _this.loginLoading = false;
                        if(response.status == 200){
                            if(response.data.statusCode == "10000"){
                                _this.imgObj.imgPath = response.data.data;
                                let url = "";
                                if(_this.imgObj.id == ""){
                                    url = "addImagesInfo";
                                }else {
                                    url = "updateImagesInfo";
                                }
                                $util.post(url,_this.imgObj)
                                .then(function (response) {
                                    if(response.status == 200 ){
                                        if( response.data.statusCode == "10000"){
                                            _this.imgObj.id = response.data.data;
                                            $util.frontSuccMsg(_this,2,response.data.msg);
                                        }else{
                                            $util.responseMsg(_this,response.data);
                                        }
                                    }else{
                                        $util.httpErrorMsg(_this,response.data)
                                    }
                                })
                                .catch(function (error) {
                                    $util.httpErrorMsg(_this,error.data)
                                })
                            }else {
                                $util.responseMsg(_this,response.data);
                            }
                        }else{
                            $util.httpErrorMsg(_this,response.data)
                        }
                    })
                    .catch(function (error) {
                        $util.httpErrorMsg(_this,error.data)
                    })
            },
            handlecrop() {
                if(this.imgObj.name == ""){
                    $util.frontErrMsg(this,2,"请输入名称");
                    return;
                }
                let file = this.cropper.getCroppedCanvas().toDataURL();
                this.option.cropedImg = file;
                this.option.showCropedImage = true;
            },
            selectShow(id){
                let _this = this;
                let url = "queryImagesInfo";
                $util.post(url,{id:id})
                    .then(function (response) {
                        if(response.status == 200){
                            if(response.data.statusCode == "10000"){
                                _this.imgObj.name = response.data.data.fileName;
                                _this.imgObj.id = response.data.data.id;
                                _this.cropper.load(response.data.data.imgUrl);
                            }else{
                                $util.responseMsg(_this,response.data);
                            }
                        }else{
                            $util.httpErrorMsg(_this,response.data)
                        }
                    })
                    .catch(function (error) {
                        $util.httpErrorMsg(_this,error.data)
                    })
            }
        },
        activated(){
            //console.info("activated() => start");
            let fileId = this.$route.params.fileId;
            if(fileId != null && fileId !=""){
                this.selectShow(fileId);
            }
        },
        mounted() {
            this.init();
            window.onresize = () => {
                return (() => {
                    this.initImageH();
                })()
            };
        },
        created(){
            this.initImageH();
        }
    }
</script>

<style scoped>

</style>