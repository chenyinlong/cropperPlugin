<template>
    <div>
        <el-upload
            action="https://jsonplaceholder.typicode.com/posts/"
            class="avatar-uploader"
            :show-file-list="false"
            :on-success="success"
            :on-change="change"
        >
            <img v-if="imageUrl" :src="imageUrl" class="avatar"/>
            <i v-else class="el-icon-plus avatar-uploader-icon"></i>
        </el-upload>
        <el-dialog title="图片剪裁" :visible.sync="pictureVisible" append-to-body>
            <div class="cropper-content">
                <div class="cropper" style="text-align:center">
                    <vueCropper
                        ref="cropper"
                        :img="pictureOption.img"
                        :outputSize="pictureOption.size"
                        :outputType="pictureOption.outputType"
                        :info="true"
                        :full="pictureOption.full"
                        :canMove="pictureOption.canMove"
                        :canMoveBox="pictureOption.canMoveBox"
                        :original="pictureOption.original"
                        :autoCrop="pictureOption.autoCrop"
                        :fixed="pictureOption.fixed"
                        :fixedNumber="pictureOption.fixedNumber"
                        :centerBox="pictureOption.centerBox"
                        :infoTrue="pictureOption.infoTrue"
                        :fixedBox="pictureOption.fixedBox"
                    />
                </div>
            </div>
            <div slot="footer" class="dialog-footer">
                <el-button @click="cropperChangeScale(1)">放大</el-button >
                <el-button @click="cropperChangeScale(-1)">缩小</el-button >
                <el-button @click="cropperRotateLeft">左旋转</el-button >
                <el-button @click="cropperRotateRight">右旋转</el-button >
                <el-button type="primary" @click="finish('blob')">确认剪切</el-button>
            </div>
        </el-dialog>
    </div>
</template>
<script>
// import ajax from '../lib/ajax';
import { uploadImg } from '../lib/api';//action地址，自行修改

export default {

    props: {
        pictureOption: {
            type: Object,
            default: () => ({
                img: '', // 裁剪图片的地址
                info: true, // 裁剪框的大小信息
                outputSize: 1, // 裁剪生成图片的质量
                full: false, // 输出原图比例截图 props名full
                outputType: 'jpeg', // 裁剪生成图片的格式
                canMove: false, // 能否拖动图片
                original: false, // 上传图片是否显示原始宽高
                canMoveBox: true, // 能否拖动截图框
                canScale: false, // 图片是否允许滚轮缩放
                autoCrop: true, // 是否默认生成截图框
                // autoCropWidth: 400, // 默认生成截图框宽度
                // autoCropHeight: 300, // 默认生成截图框高度
                fixedBox: false, // 截图框固定大小
                fixed: true, // 是否开启截图框宽高固定比例
                fixedNumber: [5, 5] // 截图框的宽高比例
            })
        },
        geturl: {
            type: String,
            default: ''
        }
    },
    data() {
        return {
            uploadImg,
            imageUrl: "",
            pictureVisible: false
        };
    },
    methods: {
        // 放大缩小
        cropperChangeScale(num) {
            num = num || 1;
            this.$refs.cropper.changeScale(num);
        },
        // 左旋
        cropperRotateLeft() {
            this.$refs.cropper.rotateLeft();
        },
        // 右旋
        cropperRotateRight() {
            this.$refs.cropper.rotateRight();
        },
        change(file){
            this.$nextTick(() => {
                this.pictureOption.img = URL.createObjectURL(file.raw);
            });
        },
        finish(type) {
            if(type==='blob'){
                this.$refs.cropper.getCropData(blob => {
                this.imageUrl = blob;
                this.pictureVisible = false;
                this.$emit('finish', this.imageUrl);
                });
            }else{
                this.$refs.cropper.getCropBlob(data => {
                this.imageUrl = window.URL.createObjectURL(data);
                this.pictureVisible = false;
                this.$emit('finish', this.imageUrl);
                });
            }
            
        },
        convertBase64UrlToBlob(urlData) {
            const bytes = window.atob(urlData.split(',')[1]); // 去掉url的头，并转换为byte
            // 处理异常,将ascii码小于0的转换为大于0
            const ab = new ArrayBuffer(bytes.length);
            const ia = new Uint8Array(ab);
            for (let i = 0; i < bytes.length; i++) {
                ia[i] = bytes.charCodeAt(i);
            }
            return new Blob([ab], { type: 'image/jpeg' });
        },
        success(file,fileList) {
            console.log(fileList)
            this.pictureOption.img = file.data;
            this.pictureVisible = true;
            console.log(file);
        }
    }
};
</script>
<style lang="stylus" scoped>
.cropper {
    width: auto;
    height: 300px;
}
.avatar {
    width: 178px;
    height: 178px;
    display: block;
}
.avatar-uploader .el-upload {
    border: 1px dashed #d9d9d9;
    border-radius: 6px;
    cursor: pointer;
    position: relative;
    overflow: hidden;
}
.avatar-uploader .el-upload:hover {
    border-color: #409EFF;
}
.avatar-uploader-icon {
    border: 1px dashed #d9d9d9;
    font-size: 28px;
    color: #8c939d;
    width: 178px;
    height: 178px;
    line-height: 178px;
    text-align: center;
}
</style>
