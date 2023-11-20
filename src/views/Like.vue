<template>
    <div class="Like">
        <div class="navi">
            <van-nav-bar title="我的收藏" left-text="返回" left-arrow @click-left="back()" />
        </div>
        <div class="account-bg"></div>
        <div class="clearfix" v-if="likeProduct.length > 0">
            <div class="like-box">
                <div class="pro-item" v-for="(item, index) in likeProduct" :key="index">
                    <div @click="getDetail(item.pid)">
                        <img class="auto_img" :src="item.smallImg" alt="">
                        <div class="pro-name one_text">{{ item.name }}</div>
                    <div class="pro-enname one_text">{{ item.enname }}</div>
                    <div class="fl pro-price">￥{{ item.price }}</div>
                    </div>
                    

                    <div class="fr delete" :item="item" @click="removeLike(item.pid,index)" :is-remove="true">
                        <van-icon name="delete-o"></van-icon>
                    </div>

                </div>
            </div>
        </div>
        <van-empty v-else description="没有收藏" />
    </div>
</template>

<script>
import "../assets/less/cell.less"

import ProItem from "../components/ProItem.vue"
export default {
    name: "Like",
    components: {
        ProItem,
    },
    data() {
        return {
            likeProduct: [],
        }
    },
    created() {
        this.getLikeProduct();
    },
    methods: {
        removeLike(pid, index) {
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
                method: "POST",
                url: "/notlike",
                data: {
                    appkey: this.appkey,
                    pid,
                    tokenString
                },
            })
                .then(result => {
                    this.$toast.clear();
                    if (result.data.code == 700) {
                        this.$router.push({ name: "Login" });
                    } else if (result.data.code == 900) {
                        this.$toast('删除成功！');
                        this.likeProduct.splice(index, 1);
                    } else {
                        this.$toast("删除失败！")
                    }
                })
                .catch(err => {
                    this.$toast.clear();
                })
        },
        getDetail(pid) {
            this.$router.push({ name: "Detail", params: { pid } });
        },
        getLikeProduct() {
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
                method: "GET",
                url: "/findAllLike",
                params: {
                    appkey: this.appkey,
                    tokenString,
                },
            })
                .then((result) => {
                    console.log(result);
                    this.$toast.clear();
                    if (result.data.code == 700) {
                        this.$router.push({ name: "Login" });
                    } else if (result.data.code == 2000) {
                        this.likeProduct = result.data.result;
                    }
                })
                .catch((err) => {
                    this.$toast.clear();
                })

        },
        back() {
            this.$router.go(-1)
        },
    }
}
</script>