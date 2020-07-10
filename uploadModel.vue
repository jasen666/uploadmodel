<template>
  <el-form-item
    style="margin:16px auto;"
    :class="className"
    :rules="{required: needRequired, validator: this.validateFileUrl,  trigger: 'blur'}"
    v-if="visible"
    prop="fileList"
    :label="label"
  >
    <div class="bannerimg">
      <span>
        <el-upload
          ref="imgUploadCallBack"
          :file-list="fileList"
          :disabled="imgFlag"
          :accept="accept"
          :limit="limit"
          :multiple="needMultiple"
          :on-exceed="handleExceed"
          :show-file-list="showFileList&&listType=='picture-card'"
          :with-credentials="withCredentials"
          :action="Hppt+port"
          :list-type="listType"
          :before-upload="toExamine"
          :on-preview="handlePictureCardPreview"
          :on-success="handleRemoveImgSuccess"
        >
       <span v-if="clearAddButton(!imgFlag)">
         <i v-if="listType==='picture-card'" ref="upLoadAdd" @click="imgUpload"  class="el-icon-plus"></i>
           <el-button v-else-if="listType==='text'" @click="imgUpload" size="small" type="primary">点 击 上 传</el-button>
         <el-button v-else size="small" @click="imgUpload" type="primary">点 击 上 传</el-button>
       </span>
          <div slot="file" slot-scope="{file}">
            <img class="el-upload-list__item-thumbnail" :src="file.url" alt />
            <span class="el-upload-list__item-actions">
              <span class="el-upload-list__item-preview" @click="handlePictureCardPreview(file)">
                <i class="el-icon-zoom-in"></i>
              </span>
              <span class="el-upload-list__item-delete" @click="handleRemove(file)">
                <i class="el-icon-delete"></i>
              </span>
            </span>
          </div>
        </el-upload>
        <div v-if="listType=='text'&&showFileList">
          <h3 style="text-indent:5.5em">文件列表：</h3>
          <span class="fileList_jasen" v-if="fileList.length===0">暂无上传文件</span>
          <p class="fileList_jasen" v-for="(item,index) in fileList" :key="index"><span>{{item.name}}</span>
                <span class="deleteIcon_jasen"><i  @click="handleRemove(item)" class="el-icon-delete"></i></span>
              </p>
        </div>
        <el-button
        size="small"
          v-if="showClearAllBtn"
          type="warning"
          :style="clearButtonStyle"
          @click="clearFileList"
        >一 键 清 除</el-button>
        <el-dialog append-to-body :visible.sync="dialogVisible">
          <img width="100%" :src="dialogImageUrl" alt />
        </el-dialog>
      </span>
    </div>
  </el-form-item>
</template>

<script>
export default {
  data() {
    return {
      imgFlag: this.fileList.length === 0 ? false : true,
      dialogVisible: false,
      dialogImageUrl: "",
      calculagraph: null
    };
  },
  props: {
    fileList: {//上传图片的一个数组，必传
      type: Array,
      default: ()=>[]
    },
    listType:{//上传文件类型
      type:String,
      default:()=>'picture-card'
    },
    limit: {//上传图片的个数，必传
      type: Number,
      default: () => 1
    },
    needMultiple: {//是否需要多选
      type: Boolean,
      default: () =>true
    },
    withCredentials: {//支持发送 cookie 凭证信息
      type: Boolean,
      default: () => false
    },
    showFileList: {//是否显示已上传文件列表
      type: Boolean,
      default: () => true
    },
    computationRule: {//上传图片尺寸大小的校验规则
      type: String,
      default: () => "==="
    },
    accept: {//是否需要规定上传类型，默认图片类型
      type: String,
      default: () => ".jpg, .png, .gif, .jpeg,.xlsx"
    },
    label: {//上传图片的左侧内容
      type: String,
      default: () => ""
    },
    port: {//上传图片的地址,必传
      type: String,
      default: () => "/ydb/admin/manage/file/upload.json"
    },
    className: {//class，个人感觉没啥用
      type: String,
      default: () => ""
    },
    clearButtonStyle: {//自定义一键清除按钮样式
      type: String,
      default: () => "margin-top:10px"
    },
    Size: {//是否需要规定上传图片的大小
      type: Number | Boolean,
      default: () => false
    },
    putSuccessShowAddBtn: {//上传成功是否还需展示上传按钮
      type:Boolean,
      default: () => false
    },
    showClearAllBtn: {//是否显示一键清除按钮
      type: Boolean,
      default: () => true
    },
    visible: {//可传可不穿，建议传，因为当你回显的时候回有个图片清空的垃圾效果
      type: Boolean,
      default: () => false
    },
    needRequired: {//是否需要校验
      type: Boolean,
      default: () => true
    },
    Proportion: {//尺寸大小，长宽比例
      type: Boolean | Array,
      default: () => false
    },
    Hppt: {//上传的端口，本地环境，测试环境，上线环境
      type: String,
      default: () => ""
    },
    errorMessage: {//校验规则的提示内容
      type: String,
      default: () => "请上传图片文件"
    },
    fromKey: {//上传成功后，将要修改的字段
      type: String | Array,
      default: () => ""
    },
    needClipping:{//上传中是否需要大小剪辑
      type:Boolean,
      default:()=>false
    }
  },
  watch: {
    visible:{
      deep:true,
      handler:function(val){
       val===true?this.ModifiedReveal(this.imgFlag):''
      }
    }
  },
  mounted() {
     this.ModifiedReveal(this.imgFlag)
  },
  methods: {
    clearAddButton(flag){//清除上传文件按钮
     return setTimeout(() => {
        return flag
      }, 1000);
    },
    handleExceed(file,fileList){
      if(file.length>this.limit){
        this.$message.warning(`上传个数超过限制个数，最多上传${this.limit}个！！！`)
      }
    },
    ModifiedReveal(imgFlag) {//上传成功后是否显示添加按钮
      if (!this.putSuccessShowAddBtn) {
        this.$nextTick(() => {
          if (imgFlag) {
            this.$refs.upLoadAdd.parentNode.parentNode.classList.add(
              "notNeedShowAddUploadImg"
            );
          } else {
            this.$refs.upLoadAdd.parentNode.parentNode.classList.remove(
              "notNeedShowAddUploadImg"
            );
          }
        });
      }
      return !imgFlag;
    },
    toExamine(file) {
      //校验大小和校验比例
      if (this.Size) {
        //大小校验
        const fileSize = file.size / 1024 / 1024 < this.Size; // 限制小于
        if (!fileSize) {
          this.$message.error(`图片大小不可超过${this.Size}MB`);
          return false;
        }
      }

      if (this.Proportion) {
        //比例尺寸
        if (
          typeof this.Proportion[0] != "number" ||
          typeof this.Proportion[1] != "number"
        ) {
          console.error(
            "The contents of the scale array need to be passed in Numbers"
          );
          return false;
        }
        const isSize = new Promise((resolve, reject) => {
          const width = this.Proportion[0];
          const height = this.Proportion[1];
          const _URL = window.URL || window.webkitURL;
          const img = new Image();
          img.onload = () => {
            let valid = null;
            switch (this.computationRule) {
              case "===":
                valid = img.width === width && img.height === height;
                break;
              case ">=":
                valid = img.width >= width && img.height >= height;
                break;
              case "<=":
                valid = img.width <= width && img.height <= height;
                break;
            }
            valid
              ? resolve()
              : reject({
                  WIDTHFLAG: img.width <= width,
                  HEIGHTFLAG: img.height <= height,
                  width: img.width,
                  height: img.height,
                  outWidth: img.width - width,
                  outHeight: img.height - height
                });
          };
          img.src = _URL.createObjectURL(file);
        }).then(
          () => {
            return file;
          },
          obj => {
            this.$alert(
              `<div>
                <p>图片尺寸建议为：${
                  this.computationRule == "==="
                    ? "大小等于"
                    : this.computationRule == ">="
                    ? "大于等于"
                    : this.computationRule == "<="
                    ? "小于等于"
                    : ""
                }  ${this.Proportion[0]} * ${this.Proportion[1]}</p>
                <p>您的图片宽度为：${obj.width}，${
                obj.outWidth > 0 ? "超出" : "缺少"
              }：${obj.outWidth}像素</p>
                <p>您的图片高度为：${obj.height}，${
                obj.outHeight > 0 ? "超出" : "缺少"
              }：${obj.outHeight}像素</p>
                </div>`,
              "图片大小超出提示",
              {
                dangerouslyUseHTMLString: true
              }
            );
            return Promise.reject();
          }
        );
        return isSize;
      }
    },
    handleRemove(file) {
      let INdex = this.fileList.findIndex(item => item.url == file.url);
      this.fileList.splice(INdex, 1);
      this.imgFlag = this.fileList.length < this.limit ? false : true;
      this.ModifiedReveal(this.imgFlag);
      let multiple = this.fromKey.constructor === Array;
      this.$emit('deleteServeFile',file)//此时会把点击的内容发给这个自定义方发，用户可以去删除服务器对应文件
      this.$emit("setUrlPath", {
        fromKey: this.fromKey,
        url: "", //传入服务器返回数据
        multiple, //判断是否是多选
        nowIndex: multiple === true ? INdex : "" //传入当前下标
      });
    }, //清除图片
    validateFileUrl(rule, value, callback) {
      //自定义校验图片内容
      if (!this.needRequired) {
        //判断是不是必填项
        callback();
        return;
      }
      if (this.$refs.imgUploadCallBack.$data.uploadFiles.length < this.limit) {
        callback(new Error(this.errorMessage));
        return;
      }
      callback();
    },
    clearFileList() {
      //清除图片
      this.$emit("update:fileList", []);
      this.imgFlag = false;
      this.ModifiedReveal(this.imgFlag);
      this.$refs.imgUploadCallBack.$data.uploadFiles = [];
    },
    handleRemoveImgSuccess(response, file, fileList) {
      this.$message({
        type:response.success==true?'success':"warning",
        message:response.success==true?'上传成功':"上传失败"
      })
     //判断是否已经上传完毕所需图片
      this.imgFlag = fileList.length < this.limit ? false : true;
      this.ModifiedReveal(this.imgFlag);
      fileList.forEach(item => {
        if (item.response) {
          item.url = item.response.imgUrl;
          delete item.raw;
          delete item.response;
          delete item.percentage;
        }
      });
      let multiple = this.fromKey.constructor === Array; //判断是上传单个还是上传多个
      this.$emit("update:fileList", fileList);
      this.$emit("setUrlPath", {
        fromKey: this.fromKey,
        url: response.imgUrl, //传入服务器返回数据
        multiple, //判断是否是多选
        response,
        nowIndex:
          multiple === true
            ? fileList.findIndex(item => item.url === response.imgUrl)
            : "" //传入当前下标
      });
    },
    handlePictureCardPreview(file) {
      //点击查看图片
      this.dialogImageUrl = file.url;
      this.dialogVisible = true;
    },
    imgUpload() {
      if (this.fileList.length === 0) {
        this.imgFlag = false;
        this.ModifiedReveal(this.imgFlag);
      }
      if (this.imgFlag) {
        //判断是否已经上传完毕所需图片
        let nowCalculagraph = Date.parse(new Date()); //防抖
        if (nowCalculagraph - this.calculagraph > 2000) {
          this.calculagraph = nowCalculagraph;
          this.$message({
            message: "若想重新上传，请删除上个文件或图片",
            type: "warning"
          });
        }
        return;
      }
    }
  }
};
</script>

<style scoped lang='scss'>
.bannerimg {
  /deep/.el-upload-list__item-delete {
    // display: block!important;
    position: static !important;
    font-size: inherit !important;
    color: inherit !important;
  }
}
.fileList_jasen{
  text-indent: 5.5em;
  height: 40px;
  line-height: 39px;
  display: flex;
}
.deleteIcon_jasen{
  text-indent: 1em;
  cursor:pointer;
  font-size: 16px;
}
.deleteIcon_jasen:hover{
  color: red;
}
</style>