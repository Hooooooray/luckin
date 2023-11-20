<template>
    <div class="pro-item">
        <div>
            <div>
                <img class="auto_img" :src="item.smallImg" alt="">
            </div>
            <div class="pro-name one_text">{{ item.name }}</div>
            <div class="pro-enname one_text">{{ item.enname }}</div>
            <div class="clearfix">
                <div class="fl pro-price">￥{{ item.price }}</div>
                <!--  <div class="fr delete">
                    <van-icon name="cross" />
                </div> -->
            </div>
        </div>
    </div>
</template>

<script>
export default {
    name: "ProItem",

    // 组件传值
    props: {
        item: {
            type: Object,
            default() {
                return {};
            }
        }
    }
}
</script>

<style lang="less" scoped>
.pro-item {

    

    .delete {
        width: 16px;
        height: 16px;
        color: #999999;
        margin-top: 3px;

    }

}
</style>