<template>
  <div>
    <div class="menu">
      <div class="menu-nav">
        <van-search placeholder="请输入搜索关键词" shape="round" @focus="searchFocus()" />
      </div>
      <div class="menu-box">
        <div class="menu-option clearfix ">
          <van-sidebar v-model="active">
            <van-sidebar-item v-for="(item, index) in menuOption" :key="index" :title="item.title"
              @click="toMenu(index, item.type)" />
          </van-sidebar>
        </div>

        <div class="menu-content clearfix">
          <div class="menu-product clearfix">
            <div class="m-pro-item clearfix" v-for="(item, index) in productData" :key="index"
              @click="getDetail(item.pid)">
              
              <div class="hot" v-show="item.isHot">热卖</div>
              <div class="pro-img">
                <img :src="item.smallImg" class="auto_img" alt="">
              </div>
              <div class="text  clearfix">

                <div class="pro-name one_text">{{ item.name }}</div>
                <div class="pro-enname">{{ item.enname }}</div>
              </div>
              <div class="pro-price">￥{{ item.price }}</div>
            </div>
          </div>
        </div>
      </div>

    </div>
  </div>

</template>
<script>
import { Toast } from 'vant';

import "../assets/less/menu.less"
import { ref } from 'vue';

export default {

  setup() {
    const active = ref(0);
    return { active };
  },
  name: "Menu",
  data() {
    return {
      menuIndex: 0,
      menuOption: [
        {
          title: "热卖推荐",
          activeIcon: require("../assets/images/icons_03.gif"),
          inactiveIcon: require("../assets/images/icons_05.gif")
        },
        {
          title: "拿铁",
          activeIcon: require("../assets/images/icons_03.gif"),
          inactiveIcon: require("../assets/images/icons_05.gif")
        },
        {
          title: "咖啡",
          activeIcon: require("../assets/images/icons_03.gif"),
          inactiveIcon: require("../assets/images/icons_05.gif")
        },
        {
          title: "瑞纳冰",
          activeIcon: require("../assets/images/icons_03.gif"),
          inactiveIcon: require("../assets/images/icons_05.gif")
        },
        {
          title: "水果茶",
          activeIcon: require("../assets/images/icons_03.gif"),
          inactiveIcon: require("../assets/images/icons_05.gif")
        },
      ],
      productData: [],
    }
  },
  created() {
    this.getType();
  },
  methods: {
    getDetail(pid) {
      // 传值
      // 调用该方法的时候，跳转到detail页面
      this.$router.push({ name: 'Detail', params: { pid } })
    },

    toMenu(index, type) {
      if (this.menuIndex == index) {
        return;
      }
      this.menuIndex = index;
      this.getProductByType(type)
    },

    getType() {
      this.$toast.loading({
        message: "加载中...",
        forbidClick: true,
        duration: 0,
      });
      this.axios({
        method: "GET",
        url: "/type",
        params: {
          appkey: this.appkey,
        }
      })
        .then((result) => {
          this.$toast.clear();
          // console.log(result);
          if (result.data.code == 400) {
            let data = result.data.result;
            data.unshift({
              id: 5,
              type: 'isHot',
              typeDesc: '热卖推荐',
              createdAt: '2022-12-24T09:57:57.000Z',
              updatedAt: '2021-01-24T09:57:57.000Z'
            });
            //map()用于返回新数组
            this.menuOption.map((v) => {
              for (let i = 0; i < data.length; i++) {
                if (v.title == data[i].typeDesc) {
                  v.type = data[i].type;
                  break;
                }
              }
            });
            let type = this.menuOption[this.menuIndex].type;
            //调用根据商品类型获取商品的方法
            this.getProductByType(type);
          }
        })
        .catch((err) => {
          this.$toast.clear();
        })
    },
    getProductByType(type) {
      let params = {
        appkey: this.appkey,
      };
      if (type == "isHot") {
        params.key = "isHot";
        params.value = 1;
      } else {
        params.key = "type";
        params.value = type;
      }
      this.$toast.loading({
        message: "加载中...",
        forbidClick: true,
        duration: 0,
      });
      this.axios({
        method: "GET",
        url: "/typeProducts",
        params,
      })
        .then((result) => {
          this.$toast.clear();
          console.log(result);
          if (result.data.code == 500) {
            this.productData = result.data.result;
          }
        })
        .catch((err) => {
          this.$toast.clear();
        })
    },
    searchFocus() {
      this.$router.push({ name: 'Search' })
    }
  }
};
</script>

<style lang="less" scoped>
.m-icon {
  width: 32px;
  height: 32px;
}
</style>