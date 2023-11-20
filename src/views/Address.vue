<template>
    <div class="Address">
        <div class="navi">
            <van-nav-bar title="地址管理" left-text="返回" left-arrow @click-left="back()" />
        </div>
        <div v-if="list.length > 0">
            <van-address-list :list="list" default-tag-text="默认" @add="add" @edit="edit" :switchable="false" />
        </div>
        <div v-else>
            <van-empty description="没有地址" />
            <van-address-list @add="add" />
        </div>
    </div>
</template>

<script>
export default {
    name: "Address",
    data() {
        return {
            list: [],
        }
    },
    created() {
        this.getAddressData();
    },
    methods: {
        getAddressData() {
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
                url: "/findAddress",
                params: {
                    appkey: this.appkey,
                    tokenString
                }
            })
                .then(result => {
                    this.$toast.clear();
                    console.log(result)
                    if (result.data.code == 700) {
                        this.$router.push({ name: "Login" });
                    } else if (result.data.code == 20000) {
                        result.data.result.map(v => {
                            v.isDefault = Boolean(v.isDefault);
                            v.id = v.aid;
                            v.address = `${v.province}${v.city}${v.county}${v.addressDetail}`
                        })
                        this.list = result.data.result;
                    }
                })
                .catch(err => {
                    this.$toast.clear();
                })
        },
        back() {
            this.$router.go(-1)
        },
        add() {
            this.$router.push({ name: "NewAddress", });
        },
        edit(item) {
            this.$router.push({ name: "NewAddress", query: { aid: item.aid } });
        }
    }

}
</script>