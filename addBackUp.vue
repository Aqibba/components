<template>
  <div class="add-around-content">
    <div class="top-title">
      <div>商户基本信息</div>
      <div>
        <el-button type="primary" size="small" @click="subForm('ruleForm')">保存</el-button>
      </div>
    </div>
    <div class="form-content margtop30">
      <el-form ref="ruleForm" :medel="ruleForm" :rules="rules" label-width="120px" size="small">
        <el-form-item label="商户名称" prop="name">
          <el-input class="width300" v-model="ruleForm.name" placeholder="请输入商户名称"></el-input>
        </el-form-item>
        <el-form-item label="商户地址">
          <el-row>
            <el-col :span="5">
              <el-select v-model="ruleForm.city" placeholder="请选择省(市)">
                <el-option v-for="(item, index) in cityList" :key="index" :label="item.name" :value="item.value"></el-option>
              </el-select>
            </el-col>
            <el-col :span="5">
              <el-select v-model="ruleForm.pra" placeholder="请选择区(县)">
                <el-option v-for="(item, index) in cityList" :key="index" :label="item.name" :value="item.value"></el-option>
              </el-select>
            </el-col>
            <el-col :span="5">
              <el-select v-model="ruleForm.area" placeholder="请选择接到(村/镇)">
                <el-option v-for="(item, index) in cityList" :key="index" :label="item.name" :value="item.value"></el-option>
              </el-select>
            </el-col>
            <el-col :span="5">
              <el-input v-model="ruleForm.address" placeholder="请输入店铺所在地址"></el-input>
            </el-col>
          </el-row>
        </el-form-item>
        <el-form-item label="联系电话">
          <el-input class="width300" v-model="ruleForm.phone" placeholder="请输入联系电话"></el-input>
        </el-form-item>
        <el-form-item label="营业时间">
          <el-row>
            <el-col :span="5">
              <el-select v-model="ruleForm.week" placeholder="请选择营业时间范围">
                <el-option v-for="(item, index) in cityList" :key="index" :label="item.name" :value="item.value"></el-option>
              </el-select>
            </el-col>
            <el-col :span="6">
              <el-time-picker
                is-range
                arrow-control
                v-model="ruleForm.time"
                range-separator="至"
                start-placeholder="开始时间"
                end-placeholder="结束时间"
                placeholder="选择时间范围">
              </el-time-picker>
            </el-col>
          </el-row>
        </el-form-item>
        <el-form-item label="服务设施">
          <el-input class="width300" v-model="ruleForm.facility" placeholder="例如：免费停车 WIFI 包间"></el-input>
        </el-form-item>
        <el-form-item label="人均消费">
          ￥<el-input v-model="ruleForm.price" class="width200" style="display: initial; margin: 0 10px;" placeholder="请输入价格"></el-input>/人
        </el-form-item>
        <el-form-item label="商户标签维护">
          <el-row :gutter="5">
            <el-col :span="4" v-for="(item, index) in ruleForm.tabs" :key="index">
              <el-input v-model="ruleForm.tabs[index]" placeholder="请输入店铺标签"></el-input>
            </el-col>
          </el-row>
        </el-form-item>
      </el-form>
    </div>
    <div class="title margtop60">商户图片信息</div>
    <div class="form-content margtop30">
      <el-form ref="imgForm" :medel="imgForm" label-width="120px" size="small">
        <el-form-item label="店铺logo">
          <el-row>
            <el-col :span="2">
              <div class="avatar-uploader">
                <el-upload
                  :class="{hide: hide}"
                  action="#"
                  ref="avatarUpload"
                  list-type="picture-card"
                  :limit="1"
                  :on-exceed="handleExcced"
                  :on-change="beforeLoad"
                  :auto-upload="false">
                  <i slot="default" class="el-icon-plus"></i>
                  <div slot="file" slot-scope="{file}" class="img-logo">
                    <img class="el-upload-list__item-thumbnail" :src="imageUrl" />
                    <span class="el-upload-list__item-actions">
                      <span class="el-upload-list__item-preview" @click="handlePictureCardPreview(file)">
                        <i class="el-icon-zoom-in"></i>
                      </span>
                      <span v-if="!disabled" class="el-upload-list__item-delete" @click="handleRemove(file)">
                        <i class="el-icon-delete"></i>
                      </span>
                    </span>
                  </div>
                </el-upload>
              </div>
            </el-col>
            <el-col :span="12">
              <div style="color: #999">Tips: 请上传150*150像素的正方形图片，图片大小请控制在100k以内</div>
            </el-col>
          </el-row>
        </el-form-item>
        <el-form-item label="首页轮播图维护">
          <el-row>
            <el-col>
              <el-upload 
                class="upload-demo"
                action="#"
                ref="bannerImage"
                :multiple="true"
                :show-file-list="false"
                :disabled="uploadDisabled"
                :on-change="bannerBeforeLoad"
                :auto-upload="false">
                <el-button size="small" type="primary">
                  <i class="el-icon-upload2"></i>
                  上传图片
                </el-button>
              </el-upload>
              <div style="margin: 10px; color: #999">Tips：请上传750*422像素长方形图片，图片大小控制在300k以内。横向拖动图片组，可以调整Banner的显示顺序。</div>
            </el-col>
          </el-row>
          <el-row>
            <el-col>
              <draggable
                class="list-group"
                v-model="dragList"
                v-bind="dragOptions"
                @start="drag = true"
                @end="drag = false">
                <div v-for="(item, index) in dragList" :key="index">
                  <div class="img-warp" :class="{'banner-img': isHover}">
                    <img :src="item.imgUrl" style="width: 100%; height: 100%; display:block; border-radius: 5px;" />
                    <div class="modalBlank">
                      <div @click="zoomBtnClick(item.imgUrl, index)" style="cursor: pointer"><i class="el-icon-zoom-in"></i></div>
                      <div @click="deleBtnClick(item, index)" style="cursor: pointer"><i class="el-icon-delete"></i></div>
                    </div>
                  </div>
                  <el-input style="margin-bottom: 10px; width: 188px" placeholder="跳转URL" v-model="item.input"></el-input>
                </div>
              </draggable>
            </el-col>
          </el-row>
        </el-form-item>
      </el-form>
      <el-dialog :visible.sync="dialogVisible">
        <img width="100%" :src="dialogImageUrl" />
      </el-dialog>
    </div>
  </div>
</template>

<script>
import draggable from "vuedraggable";

let phoneRegx = (rule, value, callback) => {
  let regx1 = /^1[3456789]\d{9}$/;
  let regx2 = /^(\+\d{1,3})(\s{1}[1-9]\d{1,2}\s{1})(\d{7,8})$/;
  if (!regx2.test(value) || !regx1.test(value)) {
    callback(new Error('请填写正确的联系方式'));
  }
}

export default {
  name: "addAround",
  components: {
    draggable
  },
  data() {
    return {
      ruleForm: {
        name: "",
        city: "",
        pra: "",
        area: "",
        address: "",
        phone: "",
        week: "",
        time: "",
        facility: "",
        price: "",
        tabs: Array(5)
      },
      rules: {
        name: [ 
          { required: true, message: "商户名称不能为空", trigger: 'blur' } 
        ],
        phone: [
          { validator: phoneRegx }
        ]
      },
      imgForm: {
        logo: {},
        bannerImage: []
      },
      cityList: [
        {
          name: "bj",
          value: "bj"
        },
        {
          name: "sh",
          value: "sh"
        },
        {
          name: "cq",
          value: "cq"
        }
      ],
      imageUrl: "",
      imgUrl: "",
      dialogImageUrl: "",
      dialogVisible: false,
      disabled: false,
      uploadDisabled: false,
      hide: false,
      isHover: false,
      dragList: [
        {
          file: {},
          imgUrl: require("@/assets/imgBanner.png"),
          input: ""
        },
        {
          file: {},
          imgUrl: require("@/assets/imgBanner.png"),
          input: ""
        },
        {
          file: {},
          imgUrl: require("@/assets/imgBanner.png"),
          input: ""
        },
        {
          file: {},
          imgUrl: require("@/assets/imgBanner.png"),
          input: ""
        },
        {
          file: {},
          imgUrl: require("@/assets/imgBanner.png"),
          input: ""
        }
      ]
    }
  },
  updated() {
  },
  methods: {
    subForm(formName) {
      const tabStr = this.ruleForm.tabs;
      let newTab = tabStr.toString()
      let form = JSON.parse(JSON.stringify(this.ruleForm));
      form.tabs = newTab;
      let subForm = new Object;
      Object.assign(subForm, form, this.dragList);
      // Object.assign(subForm, this.dragList);
      console.log(subForm);
      
    },
    // logo上传
    beforeLoad(file) {
      var _this = this;
      let isSize = file.raw.size / 1024 < 100;
      if (!isSize) {
        this.$message({
          type: "error",
          message: "上传图片不可超过100k，还请按需上传",
          center: type
        })
        return;
      }
      new Promise(function(resolve, reject) {
        const width = 300;
        const height = 300;
        let _url = window.URL || window.webkitURL;
        let img = new Image();
        img.onload = function () {
          let valid = img.width === width && img.height === height;
          valid ? resolve() : reject();
        };
        img.src = _url.createObjectURL(file.raw);
      }).then(() => {
        _this.imageUrl = URL.createObjectURL(file.raw);
        _this.hide = true;
        this.imgForm.logo = file;
        return file.raw;
      }, () => {
        this.$message.error("尺寸错误，请按需上传");
        _this.$refs.avatarUpload.clearFiles();
        return Promise.reject();
      }).catch(() => {
        console.log("尺寸错误")
      })
    },
    handleRemove(file, fileList) {
      this.$confirm(`是否删除图片：${file.name}?`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      }).then(() => {
        this.$refs.avatarUpload.clearFiles();
        this.hide = false;
        this.$message({
          type: "success",
          message: "删除成功",
          center: true
        });
      }).catch(() => {
        this.$message({
          type: "info",
          message: "已取消",
          center: true
        });
      });
    },
    handlePictureCardPreview(file) {
      this.dialogImageUrl = file.url;
      this.dialogVisible = true;
    },
    handleExcced(file, fileList) {
      this.$message({
        type: "error",
        message: "店铺logo只能上传一张",
        center: true
      });
    },
    // 轮播上传
    bannerBeforeLoad(file, fileList) {
      let _this = this;
      let isSize = file.raw.size / 1024 < 300;
      if (!isSize) {
        this.$message({
          type: "error",
          message: "上传图片不可超过100k，还请按需上传",
          center: type
        })
        return;
      }
      new Promise(function(resolve, reject) {
        const width = 775;
        const height = 442;
        let _url = window.URL || window.webkitURL;
        let img = new Image();
        img.onload = function () {
          let valid = img.width === width && img.height === height;
          valid ? resolve() : reject();
        };
        img.src = _url.createObjectURL(file.raw);
      }).then(() => {
        _this.isHover = true;
        let bannerImg = {
          file: file,
          imgUrl: URL.createObjectURL(file.raw),
          input: ""
        }
        _this.dragList.unshift(bannerImg);
        _this.dragList.pop();
        _this.imgForm.bannerImage.push(file);
        return file.raw;
      }, () => {
        this.$message.error("尺寸错误，请按需上传");
        _this.$refs.bannerImage.clearFiles();
        return Promise.reject();
      }).catch(() => {
        console.log("尺寸错误")
      })
    },
    zoomBtnClick(val) {
      this.dialogImageUrl = val;
      this.dialogVisible = true;
    },
    deleBtnClick(item, val) {
      this.$confirm(`是否删除图片：${item.file.name}?`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      }).then(() => {
        this.dragList.splice(val, 1);
        this.dragList.push({
          file: {},
          imgUrl: require("@/assets/imgBanner.png"),
          input: ""
        })
        this.$message({
          type: "success",
          message: "删除成功",
          center: true
        });
      }).catch(() => {
        this.$message({
          type: "info",
          message: "已取消",
          center: true
        });
      });
    }
  },
  computed: {
    dragOptions() {
      return {
        animation: 200,
        group: "description",
        disabled: false,
        ghostClass: "ghost"
      };
    },
    dragList1() {
      return this.dragList
    }
  }
}
</script>

<style lang="less" scoped>
.margtop30 {
  margin-top: 30px;
}
.margtop60 {
  margin-top: 60px;
}
.width300 /deep/ input {
  width: 300px;
}
.width200 /deep/ input {
  width: 200px;
}
.add-around-content {
  padding: 24px;
}
.top-title {
  display: flex;
  justify-content: space-between;
  align-items: baseline;
}
.avatar-uploader {
  display: inline-block; 
}
.avatar-uploader /deep/ .el-upload--picture-card {
  width: 74px;
  height: 74px;
  line-height: 87px
}
.hide /deep/ .el-upload--picture-card {
  display: none
}
.avatar-uploader /deep/ .el-upload {
  border: 1px dashed #000;
  border-radius: 5px;
  cursor: pointer;
  overflow: hidden;
}
.avatar-uploader /deep/ .el-upload-list--picture-card .el-upload-list__item {
  width: 74px;
  height: 74px;
}
.avatar-uploader /deep/ .el-upload-list--picture-card .el-upload-list__item-actions {
  font-size: 14px;
}
.avatar-uploader .el-upload:hover {
  border-color: #409eff;
}
.avatar-uploader-icon {
  font-size: 28px;
  color: #8c939d;
  width: 100px;
  height: 100px;
  line-height: 100px;
  text-align: center;
}
.avatar {
  width: 100px;
  height: 100px;
  display: block;
}
.img-warp {
  position: relative;
  width: 188px; 
  height: 106px; 
  border: 1px solid #999; 
  border-radius: 5px; 
  margin-bottom: 10px
}
.modalBlank {
  display: none;
}
.banner-img:hover .modalBlank {
  display: block;
  position: absolute;
  top: 0;
  display: flex;
  justify-content: space-around;
  align-items: center;
  width: 188px; 
  height: 106px; 
  border-radius: 5px; 
  background-color: rgba(0, 0, 0, .3);
  i {
    font-size: 20px;
  }
}
.list-group {
  display: flex;
  justify-content: space-between
}
</style>