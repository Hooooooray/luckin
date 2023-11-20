<template>
    <div class="login">
        <van-nav-bar right-text="首页" @click-right="goHome('Home')">
            <template #left>
                <div class="nav-left">
                    <div class="logo">
                        <img class="auto_img" src="../assets/images/home_active.png" alt="">
                        <div class="logo-text">Luckin Coffee</div>
                    </div>
                </div>
            </template>
        </van-nav-bar>
        <div class="login-box">
            <div class="welcome">欢迎回来！</div>
            <div class="welcome-en">Please login to your accounts</div>
            <div class="form-box">
                <van-form>
                    <van-cell-group>
                        <van-field v-model="userInfo.phone" name="手机号" label="手机号" placeholder="手机号" />
                        <van-field v-model="userInfo.password" :type="isRegisterInfo ? 'password' : 'text'" name="密码"
                            label="密码" placeholder="密码(6-16位)" :right-icon="isRegisterInfo ? 'closed-eye' : 'eye-o'"
                            @click-right-icon="isRegisterPasswordType()" />
                    </van-cell-group>
                    <div class="forgot">
                        <div class="forgot-pwd" @click="goState('Forgot')">忘记密码？</div>
                    </div>

                    <div class="login-btn">
                        <van-button class="v-login" round block type="primary" native-type="submit" @click="login()">
                            登录
                        </van-button>
                    </div>
                    <div class="login-btn">
                        <van-button class="v-register" round block type="primary" native-type="submit"
                            @click="toggleRegisterBox()">
                            注册
                        </van-button>
                    </div>
                </van-form>
            </div>
            <!-- 注册表单 -->
            <div class="van">
                <van-popup class="register-box" closeable position="bottom" v-model="isShow">
                    <div class="register-title">注册</div>
                    <van-cell-group>
                        <van-field v-model="userRegisterInfo.phone" name="手机号" label="手机号" placeholder="手机号" />
                        <van-field v-model="userRegisterInfo.password" :type="isRegisterInfo ? 'password' : 'text'"
                            name="密码" label="密码" placeholder="密码(6-16位)"
                            :right-icon="isRegisterInfo ? 'closed-eye' : 'eye-o'"
                            @click-right-icon="isRegisterPasswordType()" />
                        <van-field v-model="userRegisterInfo.nickName" name="昵称" label="昵称" placeholder="昵称(1-10位)" />
                        <div style="margin:15px">
                            <van-button class="v-login" round block type="primary" native-type="submit"
                                @click="register()">
                                注册
                            </van-button>
                        </div>
                    </van-cell-group>
                </van-popup>
            </div>
        </div>
    </div>
</template>
<script>
import "../assets/less/login.less"
// 导入表单验证模块
import { validForm } from "../assets/js/validForm"
export default {
    name: "Login",
    data() {
        return {
            // 用户登录信息
            userInfo: {
                phone: "",
                password: "",
            },
            userRegisterInfo: {
                phone: "",
                password: "",
                nickName: "",
            },
            // 注册信息的开关
            isShow: false,
            // 密码类型
            isRegisterInfo: true,
        }
    },
    created() {
        this.register();
        this.login();
        
    },
    methods: {
        goHome(name) {
            this.$router.push({ name })
        },
        goState(name) {
            this.$router.push({ name })
        },
        // 切换密码框类型
        isRegisterPasswordType() {
            this.isRegisterInfo = !this.isRegisterInfo
        },
        // 控制注册框的显示和隐藏
        toggleRegisterBox() {
            this.isShow = !this.isShow
        },
        // 注册方法
        register() {
            // 构建表单验证信息
            let o = {
                phone: {
                    value: this.userRegisterInfo.phone,
                    errorMsg: '手机号格式不正确',
                    reg: /^1[3-9]\d{9}$/
                },
                password: {
                    value: this.userRegisterInfo.password,
                    errorMsg: '密码支持数字字母下划线组合,且必须以字母开头,6-16位',
                    reg: /^[A-Za-z]\w{5,15}$/
                },
                nickName: {
                    value: this.userRegisterInfo.nickName,
                    errorMsg: '昵称1-10位',
                    reg: /^[\w\u4E00-\u9fa5]{1,10}$/
                }
            };
            let isPass = validForm.valid(o);
            if (isPass) {
                // 复制用户的注册信息并保存，发送到接口进行注册
                let userInfo = Object.assign({}, this.userRegisterInfo);
                userInfo.appkey = this.appkey;
                this.$toast.loading({
                    message: "注册加载中...",
                    forbidClick: true,
                    duration: 0,
                });
                this.axios({
                    method: "POST",
                    url: "/register",
                    data: userInfo,
                })
                    .then((result) => {
                        this.$toast.clear();
                        // console.log(result);
                        if(result.data.code==100){
                            this.isShow=false;
                        }else{
                            this.$toast(result.data.msg);
                        }
                    })
            }
        },
        login(){
            // 构建表单信息
            let o={
                phone: {
                    value: this.userInfo.phone,
                    errorMsg: '手机号格式不正确',
                    reg: /^1[3-9]\d{9}$/
                },
                password: {
                    value: this.userInfo.password,
                    errorMsg: '密码支持数字字母下划线组合,且必须以字母开头,6-16位',
                    reg: /^[A-Za-z]\w{5,15}$/
                },
            };
            let isPass = validForm.valid(o);
            if(isPass){
                // 发起登录请求，拷贝用户输入的信息
                let userInfo = Object.assign({}, this.userInfo);
                userInfo.appkey = this.appkey;
                this.$toast.loading({
                    message: "注册加载中...",
                    forbidClick: true,
                    duration: 0,
                });
                this.axios({
                    method: "POST",
                    url: "/login",
                    data: userInfo,
                })
                .then(result=>{
                    this.$toast.clear();
                    // console.log(result)
                    if(result.data.code==200){
                        // 登陆成功
                        // 保存token在localstorge方便以后验证
                        localStorage.setItem('token',result.data.token)
                        this.$router.push({name:"My"});
                    }else{
                        this.$toast(result.data.msg);
                        
                    }
                })
            }
        }
    }
}
</script>
