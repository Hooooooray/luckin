<template>
  <div class="Account">
    <div class="navi">
      <van-nav-bar title="个人资料" left-text="返回" left-arrow @click-left="back()" />
    </div>
    <div class="account-bg"></div>
    <div class="list-box">
      <van-cell-group>
        <van-cell title="头像">
          <div class="avatar">
            <img :src="accountInfo.userImg" class="auto_img" alt="">
            <van-uploader class="uploader-box" :after-read="uploadAvatar"></van-uploader>
          </div>
        </van-cell>
        <van-cell title="用户id">{{ accountInfo.userId }}</van-cell>
        <van-cell title="手机号">{{ accountInfo.phone }}</van-cell>
        <van-cell title="昵称" >
          <van-field v-model="accountInfo.nickName" maxlength="10" input-align="right" placeholder="请输入昵称" @change="uploadNickName"></van-field>
        </van-cell>
        <van-cell title="简介">
          <van-field v-model="accountInfo.desc" maxlength="10" input-align="right" placeholder="请输入简介" @change="uploadDesc"></van-field>
        </van-cell>
      </van-cell-group>
    </div>
  </div>
</template>

<script>
import "../assets/less/cell.less"
export default {
  name: "Account",
  data() {
    return {
      accountInfo: {
        nickName: "",
        desc: "",
        phone: "",
        userId: "",
        desc: "",
      },
    };
  },
  created() {
    this.getAccountInfo()
  },
  methods: {
    uploadDesc(){
      if(!this.accountInfo.desc){
        this.$toast("简介不能为空");
      }
      let tokenString = localStorage.getItem("token")
      // console.log(tokenString)
      if (!tokenString) {
        this.$toast("请先登录");
        return this.$router.push({ name: "Login" })
      }
      this.$toast.loading({
        //提示文字
        message: "正在获取信息",
        //点击穿透
        forbidClick: true,
        //延迟
        duration: 0,
      });
      this.axios({
        method:"POST",
        url:"/updateDesc",
        data:{
          appkey:this.appkey,
          tokenString,
          desc:this.accountInfo.desc
        },
      })
      .then((result)=>{
        console.log(result);
        this.$toast.clear();
        if(result.data.code==700){
          this.$router.push({name:"Login"})
        }else{
          this.$toast(result.data.msg)
        }
      })
      .catch((err)=>{
        this.$toast.clear();
      })
    },
    uploadNickName(){
      // 判断昵称是否为空
      if(!this.accountInfo.nickName){
        this.$toast("昵称不能为空");
      }
      let tokenString = localStorage.getItem("token")
      // console.log(tokenString)
      if (!tokenString) {
        this.$toast("请先登录");
        return this.$router.push({ name: "Login" })
      }
      this.$toast.loading({
        //提示文字
        message: "正在获取信息",
        //点击穿透
        forbidClick: true,
        //延迟
        duration: 0,
      });
      this.axios({
        method:"POST",
        url:"/updateNickName",
        data:{
          appkey:this.appkey,
          tokenString,
          nickName:this.accountInfo.nickName
        },
      })
      .then((result)=>{
        console.log(result);
        this.$toast.clear();
        if(result.data.code==700){
          this.$router.push({name:"Login"})
        }else{
          this.$toast(result.data.msg)
        }
      })
      .catch((err)=>{
        this.$toast.clear();
      })
    },
    uploadAvatar(file) {
      let base64 = file.content.replace(/^data:image\/[A-Za-z]+;base64,/, '');

      let tokenString = localStorage.getItem("token")
      // 允许上传的文件类型
      let type = ['jpg', 'png', 'git', 'jpeg', 'webp'];
      // 允许最大大小1mb
      let size = 1;
      console.log(file);
      // 判断文件类型
      let fileType = file.file.type.split('/')[1];
      if (type.indexOf(fileType) === -1) {
        this.$toast(`文件类型仅支持${type.join(',')}`);
        return;
      }
      let fileSize = file.file.size / 1024 / 1024;
      console.log(fileSize)
      if (fileSize > size) {
        this.$toast(`文件最大不能超过${size}MB`);
        return;
      }
      this.$toast.loading({
        message: "正在上传",
        forbidClick: true,
        duration: 1
      })
      this.axios({
        method: "POST",
        url: "/updateAvatar",
        data: {
          appkey: this.appkey,
          tokenString: tokenString,
          imgType: fileType,
          serverBase64Img: base64
        }
      })
      .then((result)=>{
        this.$toast.clear();
        console.log(result);
        // location.reload();
        if(result.data.code==700){
          this.$router.push({name:"Login"})
        }else if(result.data.code=="H001"){
          this.accountInfo.userImg = result.data.userImg;
        }
        this.$toast(result.data.msg)
      })
      .catch((err)=>{
        this.$toast.clear();
      })
    },
    back() {
      this.$router.go(-1)
    },
    getAccountInfo() {
      let tokenString = localStorage.getItem("token")
      // console.log(tokenString)
      if (!tokenString) {
        this.$toast("请先登录");
        return this.$router.push({ name: "Login" })
      }
      this.$toast.loading({
        //提示文字
        message: "正在获取信息",
        //点击穿透
        forbidClick: true,
        //延迟
        duration: 0,
      }),
        this.axios({
          method: "GET",
          url: "/findAccountInfo",
          params: {
            appkey: this.appkey,
            tokenString: tokenString,
          }
        })
          .then((result) => {
            this.$toast.clear();
            console.log(result);
            if (result.data.code == 700) {
              this.$router.push({ name: "Login" })
            } else if (result.data.code == "B001") {
              this.accountInfo = result.data.result[0];
            }
            this.$toast(result.data.msg);


          })
          .catch((err) => {
            this.$toast.clear()
          })
    },
  }
}
</script>