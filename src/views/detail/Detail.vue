<template>
  <div id="detail">
    <detail-nav-bar @titleClick="titleClick" ref="nav"/>
    <scroll class="content" ref="scroll" :probe-type="3" @scroll="contentScroll">
      <detail-swiper :top-images="topImages"/>
      <detail-base-info :goods="goodsInfo"/>
      <detail-shop-info :shop="shopInfo"/>
      <detail-goods-info :detail-info="detailInfo" @detailGoodsLoad="detailGoodsLoad"/>
      <detail-params-info ref="params" :param-info="paramInfo"/>
      <detail-comment-info ref="comment" :comment-info="commentInfo"/>
      <goods-list ref="recommend" :goods="recommends"/>
    </scroll>
    <back-top @click.native="backClick" v-show="isShowBackTop"/>
    <detail-bottom-bar @addCart="addTopCart"/>
  </div>
</template>

<script>
import DetailNavBar from './childComps/DetailNavBar'
import DetailSwiper from './childComps/DetailSwiper'
import DetailBaseInfo from './childComps/DetailBaseInfo'
import DetailShopInfo from './childComps/DetailShopInfo'
import DetailGoodsInfo from './childComps/DetailGoodsInfo'
import DetailParamsInfo from './childComps/DetailParamsInfo'
import DetailCommentInfo from './childComps/DetailCommentInfo'
import DetailBottomBar from './childComps/DetailBottomBar'

import Scroll from 'components/common/scroll/Scroll'
import GoodsList from 'components/content/goods/GoodsList'

import {getDetail, Goods, getRecommend} from 'network/detail'
import {debounce} from 'common/utils'
import {itemListenerMixin, BackTopMixin} from 'common/mixin'

import {mapActions} from 'vuex'


export default {
  name: 'Detail',
  components: {
    DetailNavBar,
    DetailSwiper,
    DetailBaseInfo,
    DetailShopInfo,
    DetailGoodsInfo,
    DetailParamsInfo,
    DetailCommentInfo,
    DetailBottomBar,
    Scroll,
    GoodsList
  },
  mixins: [itemListenerMixin, BackTopMixin],
  data() {
    return {
      iid: null,
      topImages: [],
      goodsInfo: {},
      shopInfo: {},
      detailInfo: {},
      paramInfo: {},
      commentInfo: {},
      recommends: [],
      themeTopYs: [],
      getThemeTopY: null,
      currentIndex: 0
    }
  },
  computed: {
    
  },
  created() {
    // 1.保存传入的iid
    this.iid = this.$route.params.iid

    // 2.根据iid请求详情数据
    getDetail(this.iid).then(res => {
      // 1.获取顶部的图片轮播数据
      console.log(res);
      this.topImages = res.result.itemInfo.topImages

      // 2.创建商品的对象
      this.goodsInfo = new Goods(res.result.itemInfo, res.result.columns, res.result.shopInfo.services)

      // 3.取出店铺信息
      this.shopInfo = res.result.shopInfo

      // 4.取出详情信息
      this.detailInfo = res.result.detailInfo

      // 5.取出参数信息
      this.paramInfo = res.result.itemParams

      // 6.取出评论信息
      if(res.result.rate.cRate !== 0){
        this.commentInfo = res.result.rate.list[0]
      }

      // this.$nextTick(() => {
      //   // 根据最新的数据,对应的DOM是已经被渲染出来
      //   // 但是图片依然是没有加载完
      //   // offsetTop值不对的时候,都是因为图片的问题
      //   // this.themeTopYs = []
      //   // this.themeTopYs.push(0);
      //   // this.themeTopYs.push(this.$refs.params.$el.offsetTop);
      //   // this.themeTopYs.push(this.$refs.comment.$el.offsetTop);
      //   // this.themeTopYs.push(this.$refs.recommend.$el.offsetTop);
      // })
    })

    // 3.请求推荐数据
    getRecommend().then(res => {
      console.log(res);
      this.recommends = res.data.list
    })

    // 4.给getThemeTopY赋值
    this.getThemeTopY = debounce(() => {
      this.themeTopYs = []
      this.themeTopYs.push(0);
      this.themeTopYs.push(this.$refs.params.$el.offsetTop);
      this.themeTopYs.push(this.$refs.comment.$el.offsetTop);
      this.themeTopYs.push(this.$refs.recommend.$el.offsetTop);
      this.themeTopYs.push(Number.MAX_VALUE);
      console.log(this.themeTopYs);
    }, 100)
  },
  mounted() {
  },
  updated() {
  },
  destroyed() {
    this.$bus.$off('itemImgLoad', this.itemImgListener)
  },
  methods: {
    ...mapActions(['addCart']),
    detailGoodsLoad() {
      this.refresh()

      this.getThemeTopY()
    },
    titleClick(index) {
      // console.log(index);
      this.$refs.scroll.scrollTo(0, -this.themeTopYs[index], 100)
    },
    contentScroll(position) {
      const positionY = -position.y

      let length = this.themeTopYs.length
      for(let i = 0; i < this.themeTopYs.length-1; i++) {
        if(this.currentIndex !== i && (positionY >= this.themeTopYs[i] && positionY < this.themeTopYs[i+1])) {
          this.currentIndex = i;
          this.$refs.nav.currentIndex = this.currentIndex
        }
        // if(this.currentIndex !== i && (i < length - 1 && positionY >= this.themeTopYs[i] && positionY < this.themeTopYs[i+1]) || (i === length - 1 && positionY >= this.themeTopYs[i])) {
        //   this.currentIndex = i;
        //   // console.log(this.currentIndex);
        //   this.$refs.nav.currentIndex = this.currentIndex
        // }
      }
      // 是否显示回到顶部
      this.listenShowBackTop(position)
    },
    addTopCart() {
      // 1.获取购物车需要展示的商品信息
      const product = {}
      product.image = this.topImages[0]
      product.title = this.goodsInfo.title
      product.desc = this.goodsInfo.desc
      product.price = this.goodsInfo.realPrice
      product.iid = this.iid

      // 2.将商品添加到购物车里
      // this.$store.dispatch('addCart', product).then(res => {
      //   console.log(res);
      // })
      this.addCart(product).then(res => {
        // this.show = true
        // this.message = res

        // setTimeout(() => {
        //   this.show = false
        //   this.message = ''
        // }, 1500);

        this.$toast.show(res, 1500)
        console.log(res);
      })
    }
  }
}
</script>

<style scoped>
  #detail {
    height: 100vh;
    background-color: #fff;
    position: relative;
    z-index: 1;
  }
  .content {
    background-color: #fff;
    height: calc(100% - 102px);
  }
</style>