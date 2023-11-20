<template>
  <div class="my">
    <div class="bg-box">
      <div class="my-bg" :style="{ backgroundImage: `url('${userInfo.userBg}')` }">
        <div class="uploader-box">

          <van-uploader :after-read="uploadBg" />

        </div>


      </div>

    </div>
    <div class="my-box">
      <div class="my-info">
        <div class="my-img">
          <img :src="userInfo.userImg" class="auto_img" alt="">
        </div>
        <div class="user-info">
          <div class="user-name">{{ userInfo.nickName }}</div>
          <div class="desc">{{ userInfo.desc == '' ? "这个人很懒，什么都没有留下" : userInfo.desc }}</div>
        </div>
      </div>
      <div class="cell-list">
        <div class="cell-list-item" v-for="(item, index) in listData" :key="index">
          <van-cell :title=item.title is-link @click="getList(item.name)" />
        </div>
      </div>

    </div>
  </div>
</template>
<script>
import "../assets/less/my.less"
export default {
  // setup() {
  //   const afterRead = (file) => {

  //     console.log(file);
  //   };

  //   return {
  //     afterRead,
  //   };
  // },
  name: "My",
  data() {
    return {
      userInfo: {},
      listData: [
        {
          title: "个人资料",
          name: "Account"
        },
        {
          title: "我的订单",
          name: "Order"
        },
        {
          title: "我的收藏",
          name: "Like"
        },
        {
          title: "地址管理",
          name: "Address"
        },
        {
          title: "安全中心",
          name: "Secure"
        },
      ]
    }
  },
  created() {
    this.getUserInfo();
  },
  methods: {
    uploadBg(file) {
      let base64 = file.content.replace(/^data:image\/[A-Za-z]+;base64,/,'');

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
        url: "/updateUserBg",
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
        location.reload();
      })
    },
    getList(name) {
      this.$router.push({ name })
    },
    getUserInfo() {
      let tokenString = localStorage.getItem("token")
      // console.log(tokenString)
      this.$toast.loading({
        //提示文字
        message: "加载中...",
        //点击穿透
        forbidClick: true,
        //延迟
        duration: 1,
      }),
        this.axios({
          method: "GET",
          url: "/findMy",
          params: {
            appkey: this.appkey,
            tokenString: tokenString,
          }
        })
          .then((result) => {
            if (result.data.code != "A001") {
              this.$router.push({ name: "Login" })
            }
            this.$toast.clear();
            // console.log(result);
            if (result.data.code == "A001") {
              this.userInfo = result.data.result[0];
            };
            // console.log(this.userInfo)
          })
          .catch((err) => {
            this.$toast.clear()
          })
    },

  }

};
</script>