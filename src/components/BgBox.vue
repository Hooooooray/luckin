<template>
    <div class="bg-box">
        <div class="bg"></div>
        <div class="bg-content"></div>
        <slot></slot>
    </div>
</template>

<script>
    export default{
        name:"BgBox",
    }
</script>

<style lang="less" scoped>
.bg{
    height: 150px;
    background-color: #fff;
}
.bg-content{
    margin: 0 10px;
    position: relative;
    top:-25px;
    background-color: #fff;
    border-top-left-radius: 10px;
    border-top-right-radius: 10px;
    padding: 10px;
    overflow: hidden;
    
}
</style>