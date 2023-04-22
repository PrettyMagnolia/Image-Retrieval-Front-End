<template>
  <div class="home">
    <el-upload
      action=""
      list-type="picture-card"
      :auto-upload="false"
      :on-change="onUploadChange"
      :on-remove="handleRemove"
      :on-preview="handlePictureCardPreview"
      :class="{ disabled: fileList.length == 1 }"
      :file-list="fileList"
    >
        <i slot="default" class="el-icon-plus"></i>
        <div class="el-upload__tip" slot="tip">只能上传jpg/png文件，且不超过500kb</div>
        <div slot="file" slot-scope="{file}">
          <img
            class="el-upload-list__item-thumbnail"
            :src="file.url" alt=""
          >
          <span class="el-upload-list__item-actions">
            <span
              class="el-upload-list__item-preview"
              @click="handlePictureCardPreview(file)"
            >
              <i class="el-icon-zoom-in"></i>
            </span>
            <span
              v-if="!disabled"
              class="el-upload-list__item-delete"
              @click="handleRemove(file)"
            >
              <i class="el-icon-delete"></i>
            </span>
          </span>
        </div>
    </el-upload>
    <el-dialog :visible.sync="dialogVisible">
      <img width="100%" :src="dialogImageUrl" alt="">
    </el-dialog>
    <el-button type="primary" style="margin: 10px;" @click="onClickSearch">搜索</el-button>
    <!-- 搜索结果 -->
    <div v-if="showResult">
      <h4>共搜索到 <span>{{imageList.length}}</span> 张图片</h4>
      <el-row :gutter="60">
        <el-col :span="8" v-for="item in imageList" :key="item">
          <el-card :body-style="{ padding: '8px',}">
            <img :src=item
                 class="image">
            <div style="padding: 14px;">
              <span> 相似的图片</span>
              <div class="bottom clearfix">
                <time class="time">{{ item.slice(item.indexOf('im')) }}</time>
                <el-button type="warning" class="button" v-on:click="handleFavorites(item)" icon="el-icon-star-off" circle></el-button>
              </div>
            </div>
          </el-card>
        </el-col>
      </el-row>
    </div>

  </div>
</template>

<script>

import axios from "../../api/axios";

export default {
  data() {
    return {
      dialogImageUrl: '',
      dialogVisible: false,
      disabled: false,
      fileList: [],
      imageList: [],
      showResult: false
    };
  },
  methods: {
    // 文件状态改变时的钩子，添加文件、上传成功和上传失败时都会被调用
    onUploadChange(file, fileList) {
      console.log("onUploadChange", file);
      this.fileList.push(file);
      const isIMAGE =
        file.raw.type === "image/jpeg" ||
        file.raw.type === "image/png" ||
        file.raw.type === "image/gif";
      const isLt1M = file.size / 1024 / 1024 < 1;

      if (!isIMAGE) {
        this.$message.error("上传文件只能是图片格式!");
        // 从文件列表中删除最后一个元素
        this.fileList.pop();
        return false;
      }
      if (!isLt1M) {
        this.$message.error("上传文件大小不能超过 1MB!");
        // 从文件列表中删除最后一个元素
        this.fileList.pop();
        return false;
      }
      console.log(fileList[0]);
    },
    handleRemove(file) {
      console.log(file);
      this.fileList = [];
    },
    handlePictureCardPreview(file) {
      this.dialogImageUrl = file.url;
      this.dialogVisible = true;
    },
    onClickSearch() {
      if (this.fileList.length != 0) {
        axios.request(
          {
            headers: {
              "Content-Type": "multipart/form-data",
            },
            url: "http://localhost:5000/imgUpload",
            method: "post",
            data: this.fileList[0],
          }
        )
        .then(res => {
          console.log("返回数据", res);

          let keys = Object.keys(res.data);
          this.imageList = [];
          for (let i = 0; i < keys.length; i++) {
            this.imageList.push("http://localhost:5000" + res.data[keys[i]].replace("\\","/"))
          }
          console.log(this.imageList);
          this.showResult=true;

        })
        .catch(err => {
          console.log("发生错误", err);
        })
      }
    }
  }
}
</script>


<style>
.disabled .el-upload--picture-card {
  display: none !important;
}

</style>
