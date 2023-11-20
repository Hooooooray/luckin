<template>
  <div class="shopbag">
    <van-nav-bar class="navi" title="购物袋" left-text="返回" left-arrow @click-left="back()" @click-right="isEdit = !isEdit">
      <template #right>
        <div v-if="shopbagData.length > 0">{{ isEdit ? '完成' : '编辑' }}</div>
      </template>
    </van-nav-bar>

    <van-empty v-if="shopbagData.length == 0" description="购物袋空空如也" />

    <div class="shopbag" v-else>
      <div class="shopbag-bg">
      </div>
      <van-list v-model:loading="loading" :finished="finished" finished-text="没有更多了" @load="loadData">
        <!-- <van-cell v-for="item in list" :key="item" :title="item" /> -->
        <div class="shopbag-product">
          <div class="shopbag-item" v-for="(item, index) in shopbagData" :key="item.sid">
            <div class="cell-item">
              <van-swipe-cell right-width:60px>
                <div class="clearfix cell-item-box">
                  <div class="fl check-box">
                    <van-checkbox v-model="item.isChecked" @change="simpleSelect"  />
                  </div>
                  <div class="fl pro-info-box">
                    <div class="fl pro-img">
                      <img class="auto_img" :src="item.small_img" alt="">
                    </div>

                  </div>
                  <div class="fl pro-info">
                    <div class="pro-info-text">
                      <div class="clearfix names">
                        <div class="fl pro-name">{{ item.name }}</div>
                        <div class="fl pro-rule">{{ item.rule }}</div>

                      </div>
                      <div class="pro-enname">{{ item.enname }}</div>
                    </div>
                    <div class="pro-count-box">
                      <div class="fl pro-price">¥{{ item.price }}</div>
                      <div class="fr step-box">
                        <van-stepper v-model="item.count" theme="round" button-size="20" disable-input @change="changeCount(item)"></van-stepper>
                      </div>
                    </div>

                  </div>

                </div>
                <template #right>
                  <van-button @click="removeOne(index, item.sid)" square type="danger" text="删除"
                    class="delete-button"></van-button>
                </template>
              </van-swipe-cell>

            </div>
          </div>
        </div>
      </van-list>
    </div>

    <div>
      <van-submit-bar v-if="!isEdit" :price="total" button-text="提交订单" button-color="#0c34ba" @submit="commit">
        <van-checkbox class="check-box" v-model="isAllChecked" @click="allSelect">全选</van-checkbox>
      </van-submit-bar>
      <van-submit-bar v-else button-text="删除选择" button-color="ED1C24" @submit="removeSelected">
        <van-checkbox class="check-box" v-model="isAllChecked" @click="allSelect">全选</van-checkbox>
      </van-submit-bar>
    </div>

  </div>
</template>
<script>
import "../assets/less/shopbag.less"
export default {
  name: "Shopbad",
  data() {
    return {
      // 是否处于编辑状态
      isEdit: false,
      // 是否选中
      isCheck: false,
      // 购物袋商品数量
      count: 2,
      // 是否全选
      isAllChecked: false,
      // 懒加载是否加载
      loading: true,
      // 是否全部加载完成数据
      finished: false,
      // 所有购物袋数据
      allShopbagData: [],
      // 购物袋的数量
      shopbagData: [],
      // 下拉加载的数量
      dataCount: 8,
      // 开哦截取购物袋数据的位置
      startIndex: 0,
      // 订单总金额
      total: 0
    }
  },
  created() {
    this.getShopbagData();
  },
  methods: {
    isSelectProduct(msg) {
      for (let i = 0; i < this.shopbagData.length; i++) {
        if (this.shopbagData[i].isChecked) {
          return true;
        }
      }
      return false;
    },
    removeSelected() {
      let isHas = this.isSelectProduct();
      if (!isHas) {
        this.$toast("请选择要删除的商品");
        return;
      }
      let sids = [];
      this.shopbagData.map((v) => {
        if (v.isChecked) {
          sids.push(v.sid);
        }
      });
      let tokenString = localStorage.getItem("token")
      // console.log(tokenString)
      if (!tokenString) {
        this.$toast("请先登录");
        return this.$router.push({ name: "Login" })
      }
      this.$toast.loading({
        message:"加载中...",
        forbidClick:true,
        duration:0,
      })
      this.axios({
        method: "POST",
        url: "/deleteShopcart",
        data: {
          appkey: this.appkey,
          tokenString,
          sids: JSON.stringify([sids]),
        }
      })
        .then((result) => {
          console.log(result);
          this.$toast.clear();
          if (result.data.code == 700) {
            this.$router.push({ name: "Login" })
          } else if (result.data.code == 7000) {
            for (let i = 0; i < this.shopbagData.length; i++) {
              if (this.shopbagData[i].isChecked) {
                this.shopbagData.splice(i, 1);
                i--
              }
            }
            this.sum();
          }
          this.$toast(result.data.msg)
        })
        .catch((err) => {
          this.toast.clear();
        })
    },
    changeCount(item) {
      let tokenString = localStorage.getItem("token")
      // console.log(tokenString)
      if (!tokenString) {
        this.$toast("请先登录");
        return this.$router.push({ name: "Login" })
      }
      this.axios({
        method:"POST",
        url:"modifyShopcartCount",
        data:{
          appkey:this.appkey,
          tokenString,
          sid:item.sid,
          count:item.count,
        },
      })
      .then((result)=>{
        this.$toast.clear();
        console.log(result);
        if(result.data.code==700){
          this.$router.push({name:"Login"});
        }else if(result.data.code==6000){
          if(item.isChecked){
            this.sum();
          }
        }else{
          this.$toast(result.data.msg)
        }
      })
      .catch((err)=>{
        this.$toast.clear();
      })
    },
    sum() {
      this.total = 0;
      this.shopbagData.map((v)=>{
        if (v.isChecked) {
          this.total += v.price * v.count;
        }
      })
      this.total *= 100;
    },

    commit() {
      let isHas = this.isSelectProduct();
      if (!isHas) {
        this.$toast("请选择商品");
        return;
      }
      let sids=[];
      this.shopbagData.map((v)=>{
        if(v.isChecked){
          sids.push(v.sid);
        }
      })
      this.$router.push({name:"Pay",query:{sids:sids.join('-')}});
    },

    // 全选
    allSelect() {
      this.shopbagData.map((v) => {
        v.isChecked = this.isAllChecked;
      });
      this.sum();
    },
    // 单选
    simpleSelect() {
      this.sum();
      for (let i = 0; i < this.shopbagData.length; i++) {
        if (!this.shopbagData[i].isChecked) {
          this.isAllChecked = false;
          return;
        }
      }
      this.isAllChecked = true;
    },
    // 单个删除
    removeOne(index, sid) {
      let tokenString = localStorage.getItem("token")
      // console.log(tokenString)
      if (!tokenString) {
        this.$toast("请先登录");
        return this.$router.push({ name: "Login" });
      }
      this.$toast.loading({
        message:"加载中",
        forbidClick:true,
        duration:0,
      })
      this.axios({
        method: "POST",
        url: "/deleteShopcart",
        data: {
          appkey: this.appkey,
          tokenString,
          sids: JSON.stringify([sid])
        }
      })
        .then((result) => {
          console.log(result);
          this.$toast.clear();
          if (result.data.code == 700) {
            this.$router.push({ name: "Login" });
          } else if (result.data.code == 7000) {
            this.shopbagData.splice(index, 1);
            this.sum();
          }
          this.$toast(result.data.msg);
        })
        .catch((err) => {
          this.$toast.clear;
        });
    },

    loadData() {
      setTimeout(() => {
        // 在allshopbagdata截取数据
        let data = this.allShopbagData.slice(
          this.startIndex,
          this.startIndex + this.dataCount
        );
        this.startIndex += this.dataCount;

        this.shopbagData.push(...data);

        this.isAllChecked = false;

        // 如果当前截取数据不足8条，下次就不截取数据
        if (data.length < this.dataCount) {
          // 没有数据可以加载了，就需要finished修改为true，这样就不会再执行onload事件
          this.finished = true;
        } else {
          // 每次出发onload事件的时候，都需要将loading修改为false
          this.loading = false;
        }
      }, 1500)
    },
    rightText() {
      this.isEdit = !this.isEdit
    },
    getShopbagData() {
      let tokenString = localStorage.getItem("token")
      // console.log(tokenString)
      if (!tokenString) {
        this.$toast("请先登录");
        return this.$router.push({ name: "Login" })
      }
      this.$toast.loading({
        //提示文字
        message: "加载中...",
        //点击穿透
        forbidClick: true,
        //延迟
        duration: 1,
      });
      this.axios({
        method: "GET",
        url: "findAllShopcart",
        params: {
          appkey: this.appkey,
          tokenString
        },
      })
        .then((result) => {
          this.$toast.clear();
          console.log(result);
          if (result.data.code == 700) {
            this.$router.push({ name: "Login" });
          } else if (result.data.code == 5000) {
            result.data.result.map((v) => {
              v.isChecked = false;
            });
            this.allShopbagData = result.data.result;
            // 截取数据
            // 根据开始截取位置和截取数据数量设置购物袋显示的数据
            this.shopbagData = this.allShopbagData.slice(
              this.startIndex,
              this.startIndex + this.dataCount
            );
            this.startIndex += this.dataCount;

            // 未加载的数据
            this.loading = false
          }
        })
        .catch((err) => {
          this.$toast.clear();
        })
    },
    back() {
      this.$router.go(-1);
    }
  }
};
</script>

<style>
.goods-card {
  margin: 0;
  background-color: white;
}

.delete-button {
  height: 100%;
}
</style>