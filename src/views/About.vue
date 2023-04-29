<template>
  <div class="about" v-loading.fullscreen.lock="fullscreenLoading">
    <h1>Favorite List</h1>
    <div v-if="showResult">
      <h3><span>{{showImageList.length}}</span> images were found</h3>
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
      imageList: [],
      showImageList: [],
      previewList: [],
      tagList: [],
      currentTagList: [],
      checkedTag: [],

      showResult: false,
      fullscreenLoading: true,

      checkAll: true,

      isIndeterminate: false,
    }
  },
  methods: {
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
          if (this.imageList[i].id == id) {
            // 删除图片数组的数据
            this.imageList.splice(i, 1);
            break;
          }
        }
        for (let i = 0; i < this.showImageList.length; i++) {
          if (this.showImageList[i].id == id) {
            // 删除展示图片数组的数据
            this.showImageList.splice(i, 1);
            // 删除预览图数组的数据
            this.previewList.splice(i, 1);
            break;
          }
        }
        this.tagList
      })
    },
  },
  mounted() {
    this.fullscreenLoading = true;
    console.log("收藏")
    axios.request({
      url: "http://localhost:5000/getAllLikes",
      method: "get",
    })
    .then(res => {
      console.log(res);console.log("原始图片数据", res);
      this.imageList = res.data;
      console.log("原始图片数据", this.imageList);
      for (let i = 0; i < this.imageList.length; i++) {
        this.imageList[i].image = "http://localhost:5000" + this.imageList[i].image;
        this.previewList.push(this.imageList[i].image);
        this.tagList.push(...this.imageList[i].tags);
      }
      // 标签列表去重
      this.tagList = Array.from(new Set(this.tagList));
      this.checkedTag =this.tagList;
      console.log("收藏图片数据", this.imageList);
      console.log("收藏图片数据", this.checkedTag);
      this.showImageList = this.imageList;
      this.showResult = true;
      this.fullscreenLoading = false;
    })
    .catch(err => {
      console.log(err);
    })
    
  }
}
</script>

<style scoped>
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
</style>