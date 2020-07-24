<template>
  <div id="home" class="wrapper">
    <nav-bar class="home-nav"><div slot="center">购物街</div></nav-bar>
     <tab-control :titles="['流行', '新款', '精选']"
                  @tabClick="tabClick" ref="tabControl1" 
                  class="tab-control" v-show="isTabFixed"></tab-control>
    <scroll class="content" ref="scroll" 
            :probe-type="3" @scroll="contentScroll" 
            :pull-up-load="true" @pullingUp="loadMore">
    <home-swiper :banners="banners" @swiperImageLoad='swiperImageLoad'></home-swiper>
    <recommend-view :recommends="recommends"/>
    <feature></feature>
    <tab-control :titles="['流行', '新款', '精选']"
                   @tabClick="tabClick"
                   ref="tabControl2"/>
    <goods-list :goods="showGoods"/>
    </scroll>

    <back-top @click.native="backClick" v-show="isShowBackTop"></back-top>
  </div>
</template>

<script>
import HomeSwiper from './childComps/HomeSwiper'
import RecommendView from './childComps/RecommendView'
import Feature from './childComps/Feature'

import NavBar from 'components/common/navbar/NavBar'
import Scroll from 'components/common/scroll/Scroll'

import TabControl from 'components/content/tabControl/TabControl'
import GoodsList from 'components/content/goods/GoodsList'
import BackTop from 'components/content/backTop/BackTop'

import {getHomeMultidata, getHomeGoods} from 'network/home'
import {debounce} from "@/common/utils"
import {NEW, POP, SELL, BACKTOP_DISTANCE} from "@/common/const";
import {itemListenerMixin} from '@/common/mixin'
  export default {
    name: "Home",
    components: {
      NavBar,
      HomeSwiper,
      RecommendView,
      Feature,
      TabControl,
      GoodsList,
      Scroll,
      BackTop
    },
    mixins: [itemListenerMixin],
    data() {
      return {
        banners: [],
        recommends: [],
        goods: {
          'pop': {page: 0, list: []},
          'new': {page: 0, list: []},
          'sell': {page: 0, list: []},
        },
        currentType: 'pop',
        isShowBackTop: false,
        tabOffsetTop : 0,
        isTabFixed: false,
        saveY: 0,
        
      }
    },
    activated() {
      this.$refs.scroll.scrollTo(0, this.saveY, 0)
      this.$refs.scroll.refresh()
    },
    deactivated() {
      //保存Y值
      // this.saveY = this.$refs.scroll.getCurrentY()
      //取消全局事件的监听
      this.$bus.$off('itemImgLoad', this.itemImgListener)
    },
    created() {
      //1.请求多个数据
      this.getHomeMultidata()
      this.getHomeGoods('pop') 
      this.getHomeGoods('new')
      this.getHomeGoods('sell')
      
    },
    computed: {
      showGoods() {
        return this.goods[this.currentType].list
      }
    },
 
    mounted() {
    },
    methods: {
      
      

      getHomeMultidata() {
          getHomeMultidata().then(res => {
        this.banners = res.data.banner.list;
        this.recommends = res.data.recommend.list;
      })
      },
      getHomeGoods(type) {
        const page = this.goods[type].page + 1
        getHomeGoods(type, page).then(res => {
          this.goods[type].list.push(...res.data.list)
          this.goods[type].page += 1
          
          this.$refs.scroll.finishPullUp()
      })
      },
      tabClick(index) {
        switch(index) {
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
        this.$refs.tabControl1.currentIndex = index;
        this.$refs.tabControl2.currentIndex = index;
      },
      backClick() {
        this.$refs.scroll.scrollTo(0, 0)
      },
      contentScroll(position) {
        //判断backTop是否显示
        this.isShowBackTop = -position.y > 1000
        //判断tabControl是否吸顶
        this.isTabFixed = (-position.y) > this.tabOffsetTop
      },
      loadMore() {
        this.getHomeGoods(this.currentType)
      },
      swiperImageLoad() {
        this.tabOffsetTop = this.$refs.tabControl2.$el.offsetTop
      }
    }
  }
</script>

<style scoped>
 #home {
    /*padding-top: 44px;*/
    height: 100vh;
    position: relative;
  }

  .home-nav {
    background-color: var(--color-tint);
    color: #fff;

    /* position: fixed;
    left: 0;
    right: 0;
    top: 0;
    z-index: 9; */
  }
 .tab-control {
    position: relative;
    z-index: 9;
  }
 

  .content {
    overflow: hidden;

    position: absolute;
    top: 44px;
    bottom: 49px;
    left: 0;
    right: 0;
  }
</style>
