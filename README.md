<template>
    <div class="upload-list">
        <form class="" id="fileForm" action="" enctype="multipart/form-data">
            <div class="cont t_c">
                <ul class="clearfix">
                    <li v-for="file in fileList">
                        <div class="img-file">
                            <input type="file" :name="file.proName" class="file-input" @change="onFileChange" />
                            <div class="loading">
                                <loading></loading>
                            </div>
                            <div :class="!file.proValue ? 'img-cont hide' : 'img-cont'">
                                <i @click="close" class="close"></i>
                                <img :src="file.proValue" alt="" />
                            </div>
                        </div>
                        <p class="red" v-if="file.isNeed">必填</p>
                        <p v-else>选填</p>
                        <p>{{file.proNameCn}}</p>
                    </li>
                </ul>
            </div>
        </form>
        <div class="btn btn-bottom" @click="toNext">保存</div>
    </div>
</template>
<script>
import {
    Toast
} from 'mint-ui';
import loading from '../pubilc/loading';
export default {
    components: {
        loading: loading
    },
    data() {
        return {
            maxsize: 100 * 1024,
            fileList: [],
            necessaryList: []
        }
    },
    methods: {
        // 下一步
        toNext() {
            var sendData = {}
            this.fileList.forEach((e) => {
                sendData[e.proName] = e.proValue;
            });
            if (this.checkForm(sendData)) {
                Toast({
                    message: "请上传图片",
                    position: 'bottom',
                    duration: 2000
                })
                return false;
            }
            this.saveData(sendData)
        },
        // 保存
        saveData(sendData) {
            Public.API_GET({
                url: "savePictureData",
                data: sendData,
                success: (result) => {
                    if (result.isSuccess) {
                        Public.saveApply("accessory")
                        if (Public.applyCheck() === true) {
                            this.$router.push({
                                path: '/inclusive/confirm'
                            })
                            return
                        } else if (Public.applyCheck() === "needFace") {
                            this.$router.push({
                                path: '/inclusive/apply-list'
                            })
                            return
                        } else {
                            this.$router.push({
                                path: '/inclusive/personal'
                            })
                        }
                    } else {
                        Toast({
                            message: result.message,
                            position: 'bottom',
                            duration: 2000
                        })
                    }
                }
            })
        },
        // 检查必填项
        checkForm(sendData) {
            for (var i = 0; i < this.necessaryList.length; i++) {
                if (!sendData[this.necessaryList[i]]) {
                    return true
                }
            }
            return false
        },
        // 获取上传附件信息
        getPictureData() {
            Public.API_GET({
                url: 'getPictureData',
                success: (result) => {
                    if (!result.isSuccess) {
                        Toast({
                            message: result.message,
                            position: 'bottom',
                            duration: 2000
                        })
                        return false;
                    }
                    this.fileList = result.data.pictureData;
                    for (var i = 0; i < this.fileList.length; i++) {
                        if (this.fileList[i].isNeed) {
                            this.necessaryList.push(this.fileList[i].proName)
                        }
                    }
                }
            });
        },
        clearFileValue(file) {
            file.wrap('<form></form>');
            file.parent()[0].reset();
            file.unwrap();
        },
        onFileChange(e) {
            const obj = e.target || e.dataTransfer,
                file = obj.files[0];
            if (!file) {
                return false;
            }
            if (!/image\/\w+/.test(file.type)) {
                Toast({
                    message: "请确保文件为图像类型",
                    position: 'bottom',
                    duration: 2000
                })
                return false;
            }
            this.upload(obj, file);
        },
        // 图片上传
        upload(obj, file) {
            const formData = new FormData(),
                ind = $(obj).parents("li").index(),
                load = $(obj).next(),
                imgCont = $(obj).siblings(".img-cont");
            console.log(ind);
            load.show();
            formData.append("file", file);
            var xhr = new XMLHttpRequest();
            xhr.open("POST", "/loan/fileUpload.html");
            xhr.onreadystatechange = () => {
                if (xhr.readyState == 4 && xhr.status == 200) {
                    load.hide();
                    imgCont.show();
                    this.fileList[ind].proValue = JSON.parse(xhr.responseText).url;
                }
            }
            xhr.send(formData);
        },
        // 删除图片
        close(){
            var fileImg = $(event.target).parents(".img-file"),
                file = fileImg.find(".file-input"),
                imgCont = fileImg.find(".img-cont");
            
            imgCont.hide();
            this.clearFileValue(file);

            Public.API_GET({
                url: 'deletePictureData',
                data: {
                    picClassifyName: file.attr("name"),
                    picUrl: imgCont.find("img").attr("src")
                },
                success: (result) => {
                    if (!result.isSuccess) {
                        Toast({
                            message: result.message,
                            position: 'bottom',
                            duration: 2000
                        })
                        return false;
                    }
                }
            });
        },
        init() {
            this.getPictureData();
        }
    },
    mounted() {
        this.$parent.setTitle("附件照片");
        Public.setPageInit(this.init);
        Public.pageInit();
    }
}
</script>
<style lang="scss">
@import "../../assets/inclusive.scss";
.upload-list {
    .cont {
        padding: 1rem 0 0 1rem;
        background-color: #fff;
        &+.cont {
            margin-top: 1rem;
        }
        li {
            position: relative;
            float: left;
            width: 50%;
            padding-right: 1rem;
            margin-bottom: 1rem;
            box-sizing: border-box;

            .img-file {
                position: relative;
                z-index: 0;
                height: pxToRem(330);
                border: 1px dotted $g999;
                margin-bottom: 1rem;
                background-color: $efeff4;
                &:before,
                &:after {
                    content: "";
                    position: absolute;
                    left: 50%;
                    top: 50%;
                    z-index: 0;
                    background-color: $g999;
                }
                &:before {
                    width: pxToRem(42);
                    height: 2px;
                    margin: -1px 0 0 (- pxToRem(21));
                }
                &:after {
                    width: 2px;
                    height: pxToRem(42);
                    margin: - pxToRem(21) 0 0 -1px;
                }
                input {
                    position: absolute;
                    left: 0;
                    top: 0;
                    z-index: 10;
                    width: 100%;
                    height: pxToRem(330);
                    opacity: 0;
                    outline: none;
                }
                .loading {
                    display: none;
                    position: absolute;
                    left: 0;
                    top: 0;
                    z-index: 5;
                    width: 100%;
                    height: pxToRem(326);
                    line-height: pxToRem(326);
                    background-color: $efeff4;
                }
                .img-cont {
                    position: relative;
                    z-index: 10;
                    width: 100%;
                    height: 100%;
                    &.hide{
                        display: none;
                    }
                    .close {
                        position: absolute;
                        right: -0.6rem;
                        top: -0.6rem;
                        width: 1rem;
                        height: 1rem;
                        line-height: 1rem;
                        padding: 1rem;
                        background-color: $g999;
                        border-radius: 50%;
                        transform: rotate(45deg);
                        &:before,
                        &:after {
                            content: "";
                            position: absolute;
                            z-index: 0;
                            background-color: $white;
                        }
                        &:before{
                            left: 20%;
                            width: 60%;
                            height: 2px;
                            margin-top: -1px;
                        }
                        &:after{
                            top: 20%;
                            width: 2px;
                            height: 60%;
                            margin-left: -1px;
                        }
                    }
                    img {

                        width: 100%;
                        height: 100%;
                    }
                }
            }

            p {
                font-size: pxToRem(24);
                color: $g666;
                &.red {
                    color: $red;
                }
            }
        }
    }
}
</style>
