<template>
  <div class="main">
    <!-- 二级路由 -->
    <div>
      <router-view></router-view>
    </div>
    <!-- 底部导航栏 -->
    <van-tabbar active-color="#232773" inactive-color="#646566"	 v-model="active" route>
  <van-tabbar-item replace :to="item.path"  v-for="(item,index) in tabbarData" :key="index" :name="item.path"><span>{{item.title}}</span>
  <!-- 给子组件提供一个占位符 --插槽 -->
  <template #icon="props">
    <img :src="props.active ? item.activeImg : item.inactiveImg">
  </template>
  </van-tabbar-item>
</van-tabbar>
  </div>
</template>

<script>
import { ref } from 'vue';
export default {
  name: "Main",

  data() {
    return {
      activeName: 'Home',
      tabbarData:[
        {
          title:'首页',
          icon:'wap-home-o',
          //标签名称，页面跳转路由名称
          name:'Home',
          path:'/main/home',
          //未选中和选中图标的路径
          inactiveImg:require('../assets/images/home.png'),
          activeImg:require('../assets/images/home_active.png'),
        },
        {
          title:'菜单',
          icon:'wap-menu-o',
          //标签名称，页面跳转路由名称
          name:'Menu',
          path:'/main/menu',
          //未选中和选中图标的路径
          inactiveImg:require('../assets/images/menu.png'),
          activeImg:require('../assets/images/menu_active.png'),
        },
        {
          title:'购物袋',
          icon:'wap-shopbag-o',
          //标签名称，页面跳转路由名称
          name:'Shopbad',
          path:'/main/shopbag',
          //未选中和选中图标的路径
          inactiveImg:require('../assets/images/shopbag.png'),
          activeImg:require('../assets/images/shopbag_active.png'),
        },
        {
          title:'我的',
          icon:'wap-my-o',
          //标签名称，页面跳转路由名称
          name:'My',
          path:'/main/my',
          //未选中和选中图标的路径
          inactiveImg:require('../assets/images/my.png'),
          activeImg:require('../assets/images/my_active.png'),
        },
        
      ]
    };
  },
  methods: {},

  setup() {
    const active = ref(0);
    return { active };
  },
};
</script>
<style lang="less">
</style>
