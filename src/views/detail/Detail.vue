<template>
  <div id="detail">
      <detail-nav-bar class="detail-nav"></detail-nav-bar>
      <scroll class="content" ref="scroll">
      <detail-swiper :top-images="topImages"></detail-swiper>
      <detail-base-info :goods="goods"></detail-base-info>
      <detail-shop-info :shop="shop"></detail-shop-info>
      <detail-param-info :param-info="paramInfo"/>
      <detail-goods-info :detail-info="detailInfo" @detailImageLoad="detailImageLoad"/>
      <detail-comment-info :comment-info="commentInfo"></detail-comment-info>
      <goods-list :goods="recommends"></goods-list>
      </scroll>
  </div>
</template>

<script>
import DetailNavBar from './childComps/DetailNavBar';
import DetailSwiper from './childComps/DetailSwiper';
import DetailBaseInfo from './childComps/DetailBaseInfo';
import DetailShopInfo from './childComps/DetailShopInfo'
import DetailGoodsInfo from './childComps/DetailGoodsInfo'
import DetailParamInfo from './childComps/DetailParamInfo'
import DetailCommentInfo from './childComps/DetailCommentInfo'

import Scroll from 'components/common/scroll/Scroll'
import GoodsList from 'components/content/goods/GoodsList'

import {getDetail, getRecommend, Goods, Shop, GoodsParam} from "network/detail"
import {debounce} from "@/common/utils"
import {itemListenerMixin} from '@/common/mixin'
export default {
    name: "Detail",
    components: {
        DetailNavBar,
        DetailSwiper,
        DetailBaseInfo,
        DetailShopInfo,
        DetailGoodsInfo,
        DetailParamInfo,
        DetailCommentInfo,
        Shop,
        Scroll,
        GoodsList,
    },
     mixins: [itemListenerMixin],
    data() {
        return {
            iid: null,
            topImages: [],
            goods: {},
            shop: {},
            detailInfo: {},
            paramInfo: {},
            commentInfo: {},
            recommends: [],
        
        }
    },
    created() {
        this.iid = this.$route.params.iid
        //请求详情数据
        getDetail(this.iid).then(res => {
            const data = res.result;
            //获取轮播图信息
            this.topImages = data.itemInfo.topImages
            //创建商品对象
            this.goods = new Goods(data.itemInfo, data.columns, data.shopInfo.services)
            //获取店铺信息
            this.shop = data.shopInfo
             // 5.获取参数的信息
            this.paramInfo = new GoodsParam(data.itemParams.info, data.itemParams.rule)
             // 4.保存商品的详情数据
            // this.detailInfo = data.detailInfo;
            //6.获取评论信息
            if(data.rate.cRate !== 0) {
                this.commentInfo = data.rate.list[0]
            }
        });
        //请求推荐数据
        getRecommend().then(res => {
            this.recommends = res.data.list
        });
    },
    mounted() {
       
    },
    destroyed() {
        this.$bus.$off('itemImgLoad',this.itemImgListener)
    },
    methods: {
        detailImageLoad() {
            this.newRefresh()
        }
    }
    
}
</script>

<style scoped>
 #detail {
    position: relative;
    z-index: 9;
    background-color: #fff;
    height: 100vh;
}
.detail-nav {
    position: relative;
    z-index: 9;
    background-color: #fff;
}
.content {
    height: calc(100% - 44px);
}
</style>