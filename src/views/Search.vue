<template>
    <div class="search">
        <van-nav-bar left-text="返回" left-arrow @click-left="back()" :fixed="true">
            <template #right>
                <div class="home-search">
                    <van-search shape="round" placeholder="请输入关键词" show-action @search="search" ref="search"
                        v-model="name" />
                </div>
            </template>
        </van-nav-bar>
        <BgBox>
            <div class="clearfix Proitem-box">
                <div class="fl items" v-for="(item, index) in products" :key="index" @click="getDetail(item.pid)">
                    <ProItem :item="item" ></ProItem>
                </div>
            </div>
        </BgBox>
    </div>
</template>


<script>
import "../assets/less/search.less"
// 导入组件
import BgBox from "../components/BgBox.vue";
import ProItem from "../components/ProItem.vue";

export default {
    name: "Search",
    data() {
        return {
            products: [],
            name: "",
        }
    },
    // 注册组件
    components: {
        BgBox,
        ProItem
    },
    created() {
        // 要在vue中的created操作dom，就一定要使用$nextTick回调函数
        this.$nextTick(() => {

            //获取搜索框
            let searchIpt = this.$refs.search.querySelector('[type="search"]');
            searchIpt.focus();
        });
    },
    methods: {
        getDetail(pid) {
      // 传值
      // 调用该方法的时候，跳转到detail页面
      this.$router.push({ name: 'Detail', params: { pid } })
    },
        back() {
            this.$router.go(-1)
        },
        search() {
            this.$toast.loading({
                message: "加载中...",
                forbidClick: true,
                duration: 0.
            });
            this.axios({
                method: "GET",
                url: "/search",
                params: {
                    appkey: this.appkey,
                    name: this.name,
                }
            })
                .then((result) => {
                    this.$toast.clear();
                    console.log(result);
                    if (result.data.code == 'Q001') {
                        this.products = result.data.result;

                    } else {
                        this.$toast(result.data.msg)
                    }
                })
                .catch((err) => {
                    this.$toast.clear();
                })
        }
    },


}
</script>

<style lang="less" scoped>

</style>