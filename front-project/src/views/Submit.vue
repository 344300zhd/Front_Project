<template>
  <div class="container">
    <div class="column">
      <el-button type="danger" @click="submit">下载文件(普通)</el-button>
      <div v-for="item in userList" :key="item.username">{{item.username}}</div>
    </div>
    <div class="column">
      <el-button type="success" @click="debounceSubmit" style="margin-bottom:10px">上传文件(防抖)</el-button>
      <UploadFile @fileList="getFileList"/>
    </div>
    <div class="column" >
      <el-button type="warning" @click="throttleSubmit">接口测试(节流)</el-button>
    </div>   
    <el-col :span="4">
      <el-input v-model="msg" @input="checkInput"></el-input>
      <div class="btn" :class="{isRed:red}">提交</div>
    </el-col>
  </div>
</template>

<script>
import { debounce } from "@/utils/Debounce.js";
import { throttle } from "@/utils/Throttle.js";
import { getAllUser } from "@/api/UserApi.js";
import { uploadFile} from "@/api/FileApi.js";
import {download } from "@/utils/download";
import UploadFile from "@/components/File/UploadFile";
import * as dayjs from "dayjs"
export default {
  name:"submit",
  components:{
    UploadFile
  },
  data(){
    return{
      userList:[{username:"暂无信息"}],
      fileList:[],
      nowDay:"",
      nowDate:new Date(),
      testObj:{},
      msg:"",
      red:false
    }
  },
  mounted(){
    this.nowDay = dayjs().format("YYYY-MM-DD HH:mm")
  },
  methods: {
    checkInput(val){
      if(val) this.red = true
      else this.red = false
    },
    /** 获取文件列表 **/
    getFileList(data){
      this.fileList = data.fileList
      console.log("fileList =>",this.fileList);
    },
    // 普通提交
    submit(){
      download("1642416690842")
    },
    // 防抖提交
    debounceSubmit:debounce(function(){
      if(!this.fileList.length){
        this.$message.warning("请先选择文件 !")
        return
      }
      let formData = new FormData()
      this.fileList.forEach(item => {
        formData.append("fileList",item.file)
      })
      uploadFile(formData)
      .then(res => {
        if(res.data.code === "success"){
          this.$message.success("上传成功")
        }
      })
    },1000),
    // 节流提交
    throttleSubmit:throttle(function(){
      getAllUser()
      .then(res => {
        if(res.status === 200){
          this.$message.success("用户权限获取成功!")
          this.userList = res.data
        }
      })
    },2000),
    // 提交日志
    printLog(type){
      let messageType = type == '普通' ? 'error' : type == '防抖' ? 'success' : 'warning'
      this.$message({
        type: messageType,
        message:"请求成功：" + type
      })
    },
  },
};
</script>

<style lang="scss" scoped>
.isRed{
  background: red !important;
}
.container{
  overflow: hidden;
  .column{
    margin-bottom: 10px;
  }
  .other{
    height: 500px;
  }
  .btn{
    width: 80px;
    height: 40px;
    margin-top: 20px;
    padding: 10px;
    text-align: center;
    background: #eee;
  }
}
</style>