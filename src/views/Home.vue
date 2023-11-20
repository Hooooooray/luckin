<template>
  <div class="home">
    <van-nav-bar>
      <template #left>
        <div class="home-nav">
          <div class="t1">{{ times }}</div>
          <div class="t2">{{myName}}</div>
        </div>
      </template>
      <template #right>
        <div class="home-search">
          <van-search shape="round" placeholder="请输入关键词" @focus="searchFocus()" />
        </div>
      </template>
    </van-nav-bar>
    <div class="home-content">
      <div class="banner-box">
        <van-swipe class="my-swipe" :autoplay="3000" indicator-color="white">
          <van-swipe-item v-for="(item, index) in bannerData" :key="index"><img class="auto_img" :src="item.bannerImg"
              alt="">
          </van-swipe-item>
        </van-swipe>
      </div>
    </div>
    <div class="product-box">
      <div>
        <div class="clearfix pro-title-box">
          <!-- <span class="pro-title">热卖推荐</span> -->
        </div>
        <div class="product clearfix">
          <div class="pro-item fl" v-for="(item, index) in hotProduct" :key="index" @click="getDetail(item.pid)">
            <div class="img-box">
              <img :src="item.smallImg" class="auto_img" alt="">
            </div>
            <!-- 热卖标签 -->
            <div class="hot">热卖</div>

            <div class="pro-info">
              <div class="pro-name">{{ item.name }}</div>
              <div class="pro-enname one_text">{{ item.enname }}</div>
              <div class="pro-price">￥{{ item.price }}</div>
            </div>

          </div>
        </div>
      </div>
    </div>

  </div>



  <!-- <van-grid :column-num="2">
      <van-grid-item v-for="value in 6" :key="value" icon="photo-o" text="文字" />
    </van-grid> -->
  <!-- <van-grid :gutter="10" :border="false" :column-num="2">
      <van-grid-item>
        <van-image
          src="https://fastly.jsdelivr.net/npm/@vant/assets/apple-1.jpeg"
        />
      </van-grid-item>
      <van-grid-item>
        <van-image
          src="https://fastly.jsdelivr.net/npm/@vant/assets/apple-2.jpeg"
        />
      </van-grid-item>
    </van-grid> -->

</template>



<script>
import { Toast } from 'vant';

import "../assets/less/home.less"

// 默认导出组件
export default {

  // 组件名称
  name: "Home",
  data() {
    return {
      //接收轮播图数据
      bannerData: [],
      //定义一个变量接收问候语数据
      times: "",
      hotProduct: [],
      myName:"",
    };
  },
  created() {
    // 调用getBannerData
    this.getBannerData();
    this.getDate();
    this.getHotProduct();
    this.getMy();
  },
  methods: {
    // 获取首页名字
    getMy() {
      let log = localStorage.getItem("loglevel")
      let tokenString = localStorage.getItem("token")
      console.log(tokenString)
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
            appkey:this.appkey,
            tokenString: tokenString,
          }
        })
          .then((result) => {
            this.$toast.clear();
            console.log(result);
            if(result.data.code == "A001"){
              this.myName = result.data.result[0].nickName;
            };
            // console.log(this.myName)
          })
          .catch((err)=>{
            this.$toast.clear()
          })
    },
    //动态获取时间
    getDate() {
      let hour = new Date().getHours();
      if (hour >= 0 && hour < 6) {
        this.times = "凌晨好,"
      } else if (hour >= 6 && hour < 11) {
        this.times = "早上好,"
      }
      else if (hour >= 11 && hour < 13) {
        this.times = "中午好,"
      }
      else if (hour >= 13 && hour < 18) {
        this.times = "下午好,"
      }
      else if (hour >= 18 && hour < 24) {
        this.times = "晚上好,"
      }
    },
    getBannerData() {
      this.$toast.loading({
        //提示文字
        message: "唱，跳，rap，篮球",
        //点击穿透
        forbidClick: true,
        //延迟
        duration: 0,
      });
      //发送请求
      this.axios({
        // 请求类型
        method: "GET",
        // 请求路径
        url: "/banner",
        //data:[] params:{}
        //请求参数（object）
        params: {
          appkey: this.appkey
        }
      })
        .then((result) => {
          // console.log(result);
          this.$toast.clear();
          if (result.data.code == 300) {
            this.bannerData = result.data.result;
            
          }
        })
        .catch((err) => {
          // console.log(err);
          this.$toast.clear();
        })
    },
    getHotProduct() {
      this.$toast.loading({
        message: "正在加载",
        forbidClick: true,
        duration: 0,
      });
      this.axios({
        method: "GET",
        url: "/typeProducts",
        params: {
          appkey: this.appkey,
          key: 'isHot',
          value: 1,
        }
      })
        .then((result) => {
          /* console.log(result); */
          this.$toast.clear();
          if (result.data.code == 500) {
            this.hotProduct = result.data.result;
            
          }
        })
    },
    //查看商品详情页面
    getDetail(pid) {
      // 传值
      // 调用该方法的时候，跳转到detail页面
      this.$router.push({ name: 'Detail', params: { pid } })
    },
    searchFocus() {
      this.$router.push({ name: 'Search' })
    }
  },
};
</script>
<style lang="less" scoped>

</style>
