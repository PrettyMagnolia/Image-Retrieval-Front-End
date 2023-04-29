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
        <div class="el-upload__tip" slot="tip">Only jpg/png files with a maximum size of 1Mb can be uploaded</div>
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
    <el-button type="primary" style="margin: 10px;" @click="onClickSearch" v-loading.fullscreen.lock="fullscreenLoading">Search</el-button>
    <!-- 搜索结果 -->
    <div v-if="showResult">
      <h3> <span>{{showImageList.length}}</span> images were found</h3>
      <el-card style="text-align: left;">
        <el-row>
          <el-checkbox :indeterminate="isIndeterminate" v-model="checkAll" @change="handleCheckAllChange">Select All</el-checkbox>
          <el-button style="float: right; margin-left: 5px;" size="mini" type="primary" @click="handleReset">Reset</el-button>
          <el-button style="float: right;" size="mini" type="primary" @click="handleFilter">Filter</el-button>
          
        </el-row>
        
        <div style="margin: 15px 0;"></div>
        <el-checkbox-group v-model="checkedTag" @change="handleCheckedChange">
          <el-checkbox v-for="tag in tagList" :label="tag" :key="tag">{{tag}}</el-checkbox>
        </el-checkbox-group>
      </el-card>
      


      <div>
      <!-- 用v-for循环渲染缩略图 -->
        <div class="covers" style="display: flex">
          <el-card class="cover" v-for="item in showImageList" :key="item.id">
            <div>
              <el-image :src="item.image" width="90%" class="min" alt="" :preview-src-list="previewList"></el-image>
              <div style="padding: 14px;">
                <b>img{{item.id}}</b>
                <div>
                  <el-tag v-for="tag in item.tags" :key="index" size="mini" style="margin-top: 5 px;margin-bottom: 20px; margin-left:2px">{{tag}}</el-tag>
                </div>
                
                <div class="bottom clearfix">
                  <el-button type="warning" class="button" icon="el-icon-star-off" v-if="!item.like" @click="handleLikeChange(item.id, item.like)">Favorite</el-button>
                  <el-button type="danger" class="button" icon="el-icon-star-on" v-else @click="handleLikeChange(item.id, item.like)">Cancel Favorite</el-button>
                </div>
              </div>
            </div>
          </el-card>
        </div>
      </div>
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
      showImageList: [],
      previewList: [],
      tagList: [],
      currentTagList: [],
      checkedTag: [],

      checkAll: true,
      isIndeterminate: false,

      fullscreenLoading: false,

      showResult: false,

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
      this.fullscreenLoading = true;
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
          this.imageList = res.data;
          this.previewList = [];
          this.tagList = [];
          for (let i = 0; i < this.imageList.length; i++) {
            console.log(this.imageList[i],this.imageList[i].image,this.imageList[i].image.replace("\\","/"));
            this.imageList[i].image = "http://localhost:5000" + this.imageList[i].image.replace("\\","/");
            console.log(this.imageList[i]);

            this.previewList.push(this.imageList[i].image);
            this.tagList.push(...this.imageList[i].tags);
          }
          // 标签列表去重
          this.tagList = Array.from(new Set(this.tagList));
          this.checkedTag =this.tagList;
          console.log("图片列表", this.imageList);
          console.log("标签列表", this.tagList);

          this.showImageList = this.imageList;

          this.fullscreenLoading = false;
          // 展示搜索结果
          this.showResult=true;

        })
        .catch(err => {
          console.log("发生错误", err);
        })
      }
    },

    handleCheckAllChange(val) {
      this.checkedTag = val ? this.tagList : [];
      this.isIndeterminate = false;
      this.currentTagList = this.tagList;
    },
    handleCheckedChange(value) {
      let checkedCount = value.length;
      console.log(value.length,this.tagList.length)
      this.currentTagList = value;
      this.checkAll = checkedCount === this.tagList.length;
      this.isIndeterminate = checkedCount > 0 && checkedCount < this.tagList.length;
    },
    handleFilter() {
      console.log("筛选信息", this.currentTagList);
      this.showImageList = this.imageList.filter(x => x.tags.some(item => this.currentTagList.includes(item)));
      this.previewList = [];
      for (let i = 0; i < this.showImageList.length; i++) {
        this.previewList.push(this.showImageList[i].image);
      }
      console.log(this.showImageList);
      console.log(this.previewList);
    },
    handleReset() {
      console.log("重置");
      this.showImageList = this.imageList;
      this.previewList = [];
      for (let i = 0; i < this.showImageList.length; i++) {
        this.previewList.push(this.showImageList[i].image);
      }
      this.checkAll = true;
      this.isIndeterminate = false;
      this.checkedTag = this.tagList;
    },
    handleLikeChange(id, like) {
      console.log(id, !like)
      axios.request({
        url: "http://localhost:5000/changeLike",
        method: "post",
        params: {
          id: id,
          like: !like
        }
      })
      .then(res => {
        console.log(res);
        for (let i = 0; i < this.imageList.length; i++) {
          if (this.imageList[i].id == id) this.imageList[i].like = !like;
        }
        for (let i = 0; i < this.showImageList.length; i++) {
          if (this.showImageList[i].id == id) this.showImageList[i].like = !like;
        }
      })
    },
  }
}
</script>


<style>
.disabled .el-upload--picture-card {
  display: none !important;
}
.covers{
    display: flex;
    justify-content: space-between;
    flex-wrap: wrap;
}
.cover{
    display: flex;
    
    align-items: center;
    justify-content: center;
    width: 33%;
    margin: 10px 0;
}
.min{
    border-radius: 10px;
    cursor: zoom-in;
    object-fit: fit;
}
.max{
    cursor: zoom-out;
    width: 100%;

}
.small{
    display: flex;
    justify-content: space-between;
    flex-wrap: wrap;
}
.cover-small{
    display: flex;
    justify-content: center;
    width: 10%;
    margin: 10px 0;
    opacity: 0.6;
    cursor: pointer;
}
.cover-small:hover{
    opacity: 1;
}
.active{
    display: flex;
}
.None{
    display: none;
}
.smallActive{
    opacity: 1;
}
</style>