<template>
  <div id="home">
    <nav-bar class="home-nav">
      <div slot="center">购物街</div>
    </nav-bar>

    <scroll class="contents"
            ref="scroll"
            :probe-type="3"
            @scroll="contentScroll"
            :pull-up-load="true" @pullingUp="loadMore">
      <home-swiper :banners="banners"/>
      <recommend-view :recommends="recommends"/>
      <feature/>
      <tab-control class="tab-control"
                   :titles="['流行','新款','精选']"
                   @tabClick="tabClick"/>
      <goods-list :goods="showGoods"/>
    </scroll>

<!--    native监听组件的修饰符-->
    <back-top @click.native="backClick" v-show="isShowBackTop"/>
  </div>
</template>

<script>

  import HomeSwiper from "./childComps/HomeSwiper"
  import RecommendView from "./childComps/RecommendView"
  import Feature from "./childComps/Feature";

  import NavBar from "components/common/nagbar/NavBar";
  import TabControl from "components/content/tabControl/TabControl";
  import GoodsList from "components/content/goods/GoodsList";
  import Scroll from "components/common/scroll/Scroll"
  import BackTop from "components/content/backTop/BackTop";

  import {
    getHomeMultidata,
    getHomeGoods
  } from "network/home";


  export default {
    name: 'Home',
    components: {

      HomeSwiper,
      RecommendView,
      Feature,

      NavBar,
      TabControl,
      GoodsList,
      Scroll,
      BackTop,
    },
    data() {
      return {
        message:'哈哈哈',
        banners: [],
        recommends: [],
        goods: {
          'pop': {page: 0, list: []},
          'new': {page: 0, list: []},
          'sell': {page: 0, list: []},
        },
        currentType:  'pop',
        isShowBackTop: false,
      }
    },
    computed: {
      showGoods() {
        return this.goods[this.currentType].list
      }
    },
    created() {
      //1.请求多个数据
      this.getHomeMultidata()
      //2.请求商品数据
      this.getHomeGoods('pop')
      this.getHomeGoods('new')
      this.getHomeGoods('sell')
    },
    methods: {
      /**
       * 事件监听相关方法
       */
      tabClick(index) {
        switch (index) {
          case 0:
            this.currentType = 'pop'
            break
          case 1:
            this.currentType = 'new'
            break
          case 2:
            this.currentType = 'sell'
            break
        }
      },
      backClick() {
        this.$refs.scroll.scrollTo(0, 0, 500)
      },
      contentScroll(position) {
        this.isShowBackTop = (-position.y) > 1000
      },
      loadMore(){
        //上拉加载
        console.log('loadMore');
        this.getHomeGoods(this.currentType)
      },


      /**
       * 网络请求相关的方法
       */
      getHomeMultidata() {
        getHomeMultidata().then(res => {
          this.banners = res.data.banner.list
          this.recommends = res.data.recommend.list
        })
      },
      getHomeGoods(type) {
        const page = this.goods[type].page + 1
        getHomeGoods(type, page).then(res => {
          this.goods[type].list.push(...res.data.list)
          this.goods[type].page += 1

          this.$refs.scroll.finishPullUp()
        })
      }
    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  #home {
    position: relative;
    padding-top: 44px;
    height: 100vh;
  }

  .home-nav {
    background-color: var(--color-tint);
    color: #fff;

    position: fixed;
    left: 0;
    right: 0;
    top: 0;
    z-index: 9;
  }

  .tab-control {
    position: sticky;
    top: 44px;
    z-index: 9;
  }
  .contents{
    overflow: hidden;
    height: calc(100vh - 93px);
    /*position: absolute;*/
    /*top: 44px;*/
    /*bottom: 49px;*/
    /*left: 0;*/
    /*right: 0;*/
  }
</style>
