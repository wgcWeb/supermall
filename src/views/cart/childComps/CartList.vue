<template>
  <div class="cart-list">
    <scroll class="content" ref="scroll">
      <!-- <li v-for="item in cartList">{{item}}</li> -->
      <cart-list-item v-for="(item, index) in cartList" :item-info="item"/>
    </scroll>
  </div>
</template>

<script>
import Scroll from 'components/common/scroll/Scroll'
import CartListItem from './CartListItem'

import {mapGetters} from 'vuex'

export default {
  name: 'CartList',
  components: {
    Scroll,
    CartListItem
  },
  computed: {
    ...mapGetters(['cartList'])
  },
  activated() {
    this.$refs.scroll.refresh()
  }
}
</script>

<style scoped>
  .cart-list{
    height: 100vh;
  }
  .content{
    /* position: absolute;
    top: 44px;
    bottom: 89px;
    left: 0;
    right: 0; */
    height: calc(100% - 93px - 40px);
    overflow: hidden;
  }
</style>