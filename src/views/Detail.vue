<template>
    <div class="detail">
        <van-nav-bar :fixed="true" title="商品详情" left-text="返回" left-arrow @click-left="back()" />
        <!-- @click-left="onClickLeft" -->
        <div>
            <div>
                <img :src="productDetail.large_img" class="auto_img" alt="">
            </div>
            <div class="pro-info-box">
                <div class="pro-info">
                    <div class="clearfix pro-box">
                        <div class="fl">
                            <div class="pro-name one_text">{{ productDetail.name }}</div>
                            <div class="pro-enname one_text">{{ productDetail.enname }}</div>
                        </div>
                        <div class="pro-price fr">${{ productDetail.price }}</div>
                    </div>
                    <div class="rule-box">
                        <div class="rule-item clearfix" v-for="(item, index) in productDetail.rules" :key="index">
                            <div class="rule-title fl">{{ item.title }}
                            </div>
                            <div class="fl">
                                <div class="r-item fl"
                                    :class="{ active: item.ruleIndex == i, 'large-text:': v.title.length >= 4 }"
                                    v-for="(v, i) in item.rule" :key="i" @click="toRule(item, i)">{{ v.title }}</div>
                            </div>
                        </div>
                    </div>
                    <div class="count-box ">
                        <div class="select-text">选择数量</div>
                        <div class="select-count">
                            <van-stepper v-model="product.count" theme="round" button-size="22" disable-input />
                        </div>
                    </div>
                    <div class="detail-desc">
                        <div class="detail-desc-title">商品描述</div>
                        <div class="detail-desc-content" v-for="(item, index) in productDetail.desc" :key="index">{{
                                (index + 1 + '、' + item)
                        }}</div>
                    </div>
                </div>
            </div>
            <van-goods-action color="blue">
                <van-goods-action-icon icon="bag-o" text="购物袋" :badge="bagCount" @click="goShopbag()"/>
                <van-goods-action-icon @click="toggleLike" :icon="isLike ? 'like' : 'like-o'"
                    :text="isLike ? '已收藏' : '收藏'" :color="isLike ? '#ED1C24' : '#595C5A'" />
                <van-goods-action-button @click="addShopBag()" color="#0c34ba" type="danger" text="加入购物袋" />
            </van-goods-action>
        </div>
    </div>

</template>


<script>

import "../assets/less/detail.less"
import { Toast } from 'vant';
export default {

    name: "Detail",
    data() {
        return {
            // 定义商品的pid
            pid: '',
            // 商品的数量信息存储
            product: {
                count: 1
            },
            // 购物袋的数量
            bagCount: 0,
            // 商品的详情数据
            productDetail: {},
            //是否收藏商品
            isLike: false,

            

            


        }
    },
    created() {
        this.pid = this.$route.params.pid;
        // console.log(this.pid);
        this.getProductDetail();

        this.findLike();
        // 查询用户购物袋数量
        this.shopBagCount();
        

    },
    methods: {
        goShopbag(){
            this.$router.push({name:"Shopbag"})
        },
        addShopBag() {
            let tokenString = localStorage.getItem("token")
            // console.log(tokenString)
            if (!tokenString) {
                this.$toast("请先登录");
                return this.$router.push({ name: "Login" })
            }

            let data = {
                tokenString,
                appkey: this.appkey,
                pid: this.pid,
                count: this.product.count,
            }
            let rs = [];
            this.productDetail.rules.map(v => {
                rs.push(v.rule[v.ruleIndex].title);
            })
            data.rule = rs.join('/');
            this.$toast.loading({
                //提示文字
                message: "加入购物袋中",
                //点击穿透
                forbidClick: true,
                //延迟
                duration: 1,
            });
            this.axios({
                method: "POST",
                url: "/addShopcart",
                data
            })
                .then(result => {
                    console.log.result
                    this.$toast.clear();
                    if (result.data.code == 700) {
                        this.$router.push({ name: "Login" });
                    } else if (result.data.code = 3000) {     
                            if (result.data.status == 1) {
                                this.bagCount++;
                            }
                    }
                    this.$toast(result.data.msg);
                })
                .catch((err) => {
                    this.$toast.clear();
                })
        },
        shopBagCount() {
            let tokenString = localStorage.getItem("token")
            // console.log(tokenString)
            if (!tokenString) {
                this.$toast("请先登录");
                return this.$router.push({ name: "Login" })
            }
            this.$toast.loading({
                //提示文字
                message: "加载中",
                //点击穿透
                forbidClick: true,
                //延迟
                duration: 1,
            });
            this.axios({
                method: "GET",
                url: "/shopcartCount",
                params: {
                    appkey: this.appkey,
                    tokenString,
                }
            })
                .then((result) => {
                    this.$toast.clear();
                    if (result.data.code == 4000) {
                        this.bagCount = result.data.result;
                    }
                })
        },
        // 查询用户是否收藏该商品
        findLike() {
            let tokenString = localStorage.getItem("token");
            if (!tokenString) {
                this.$router.push({ name: "Login" })
            }
            this.$toast.loading({
                message: "获取信息中",
                forbidClick: true,
                duration: 0,
            })
            this.axios({
                method: "GET",
                url: "/findlike",
                params: {
                    appkey: this.appkey,
                    pid: this.pid,
                    tokenString
                }
            })
                .then(result => {
                    console.log(result);
                    this.$toast.clear();
                    if (result.data.code == 1000) {
                        if (result.data.result.length > 0) {
                            this.isLike = true;
                        }
                    }
                })
                .catch(err => {
                    this.$toast.clear();
                })
        },
        // 收藏商品
        toggleLike() {
            let tokenString = localStorage.getItem("token");
            if (!tokenString) {
                this.$router.push({ name: "Login" })
            }
            this.$toast.loading({
                message: "获取信息中",
                forbidClick: true,
                duration: 0,
            })
            let url = this.isLike ? "/notlike" : "/like";
            this.axios({
                method: "POST",
                url,
                data: {
                    appkey: this.appkey,
                    pid: this.pid,
                    tokenString
                }
            })
                .then((result) => {
                    // console.log(result);
                    this.$toast.clear();
                    if (result.data.code == 700) {
                        this.$router.push({ name: "Login" });
                    } else if (result.data.code == 800) {
                        this.isLike = true;
                    } else if (result.data.code == 900) {
                        this.isLike = false;
                    }
                    this.$toast(result.data.msg);
                })
                .catch(err => {
                    this.$toast.clear();
                })
        },
        onClickIcon() {
            Toast('点击图标');
        },
        onClickButton() {
            Toast('点击按钮');
        },

        toRule(item, i) {
            if (item.ruleIndex == i) {
                return;
            }
            item.ruleIndex = i;
        },
        getProductDetail() {
            this.$toast.loading({
                message: "加载中",
                forbidClick: true,
                duration: 0,
            });
            this.axios({
                method: "GET",
                url: "/productDetail",
                params: {
                    appkey: this.appkey,
                    pid: this.pid
                }
            })
                .then((result) => {
                    this.$toast.clear();
                    // console.log(result);
                    if (result.data.code == 600) {
                        let data = result.data.result[0];
                        // console.log(data);
                        data.desc = data.desc.split(/\n/);
                        let rules = [];
                        let rulesData = ['tem', 'sugar', 'milk', 'cream'];
                        rulesData.map(v => {
                            let r = {};
                            r.title = data[v + '_desc'];

                            r.ruleIndex = 0;
                            r.rule = [];
                            let ruleString = data[v].split('/');
                            ruleString.map(value => {
                                if (value != "") {
                                    r.rule.push({ title: value });
                                }
                            })
                            if (r.rule.length > 0) {
                                rules.push(r);
                            }
                        })
                        data.rules = rules;
                        this.productDetail = data;
                    }
                })
                .catch((err) => {

                })
        },
        back() {
            this.$router.go(-1)
        }
    }
}
</script>


</style>