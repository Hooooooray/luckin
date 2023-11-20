<template>
    <div class="newaddress">
        <van-nav-bar :title="aid ? '编辑地址' : '新增地址'" left-text="返回" left-arrow @click-left="back()" />
        <van-address-edit :area-list="areaList" show-postal :show-delete="!!aid" show-set-default
            :area-columns-placeholder="['请选择', '请选择', '请选择']" @save="save" @delete="removeAddress"
            :address-info="addressInfo" save-button-text="保存地址" />

    </div>
</template>

<script>
import areaList from "../assets/js/area";
export default {
    name: "NewAddress",
    data() {
        return {
            aid: "",
            areaList,
            addressInfo: {},
        }
    },
    created() {
        this.aid = this.$route.query.aid;
        if (this.aid) {
            this.getAddressDataByAid();
        }
    },
    methods: {
        save(content) {
            let tokenString = localStorage.getItem("token")
            // console.log(tokenString)
            if (!tokenString) {
                this.$toast("请先登录");
                return this.$router.push({ name: "Login" })
            }
            let addressInfo = Object.assign({}, content);
            delete addressInfo.country;
            addressInfo.tokenString = tokenString;
            addressInfo.appkey = this.appkey;
            addressInfo.isDefault = Number(addressInfo.isDefault);
            this.$toast.loading({
                message: "正在获取信息",
                forbidClick: true,
                duration: 1,
            });
            if (!this.aid) {
                this.address(addressInfo, '/addAddress');
            } else {
                let isModify = false;
                for (let key in this.addressInfo) {
                    if (content[key] != this.addressInfo[key]) {
                        isModify = true;
                    }
                }
                if(isModify){
                    this.address(addressInfo, '/editAddress');
                }else{
                    return this.$toast("请填写好地址信息");
                }
            }
        },
        address(addressInfo,url){
            this.axios({
                method:"POST",
                url,
                data:addressInfo,
            })
            .then((result)=>{
                console.log(result);
                this.$toast.clear();

                if(result.data.code==700){
                    this.$router.push({name:"Login"})
                }else if(result.data.code==9000 || result.data.code==30000){
                    setTimeout(()=>{
                        this.$router.push({name:"Address"});
                    },500)
                }
                this.$toast(result.data.msg);
            })
            .catch((err)=>{
                this.$toast.clear();
            })
        },
        back() {
            this.$router.go(-1)
        },
        getAddressDataByAid() {
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
                duration: 1,
            });
            this.axios({
                method: "GET",
                url: "/findAddressByAid",
                params: {
                    appkey: this.appkey,
                    tokenString,
                    aid: this.aid,
                }
            })
                .then((result) => {
                    if (result.data.code == 700) {
                        this.$router.push({ name: "Login" });
                    } else if (result.data.code == 40000) {
                        let data = result.data.result[0];
                        data.isDefault = Boolean(data.isDefault);
                        this.addressInfo = data;
                    }
                })
                .catch((err) => {
                    this.$toast.clear();
                })
        },
        removeAddress() {
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
                duration: 1,
            });
            this.axios({
                method:"POST",
                url:"/deleteAddress",
                data:{
                    appkey:this.appkey,
                    tokenString,
                    aid:this.aid,
                },
            })
            .then((result)=>{
                this.$toast.clear();
                console.log(result);
                if(result.data.code==700){
                    this.$router.push({name:"Login"})
                }else if(result.data.code==10000){
                    setTimeout(()=>{
                        this.$router.push({name:"Address"});
                    },500);
                }
                this.$toast(result.data.msg);
            })
            .catch((err)=>{
                this.$toast.clear();
            })
        }
    }
}
</script>