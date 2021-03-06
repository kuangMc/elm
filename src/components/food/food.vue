<template>
<transition name="move">
    <div class="food" v-show="showFlag" ref="food">
        <div class="food-content">
            <div class="image-header">
                <img :src="food.image">
                <div class="back" @click="hide">
                    <i class="icon-arrow_lift"></i>
                </div>
            </div>
            <div class="content">
                <h1 class="title">{{food.name}}</h1>
                <div class="detail">
                    <span class="sell-count">月售{{food.sellCount}}份</span>
                    <span class="rating">好评率{{food.rating}}%</span>
                </div>
                <div class="price">
                  <span class="now">¥{{food.price}}</span><span v-if="food.oldPrice" class="old">¥{{food.oldPrice}}</span>
                </div>
                <div class="cartcontrol-wrapper">
                  <cartcontrol :food="food" @cart-add="cartAdd"></cartcontrol>
                </div>
                <transition name="fade">
                 <div class="buy" v-show="!food.count" @click.stop="addFirst">加入购物车</div>
                </transition>
            </div>
            <split v-show="food.info"></split>
            <div class="info" v-show="food.info">
              <h1 class="title">商品介绍</h1>
              <p class="text">{{food.info}}</p>
            </div>
            <split></split>
            <div class="rating">
              <h1 class="title">商品评价</h1>
              <ratingselect @toggle-content="toggleContent" @type-change="typeChange" :select-type="selectType" :only-content="onlyContent" :desc="desc" :ratings="food.ratings"></ratingselect>
              <div class="rating-wrapper">
                <ul v-show="food.ratings&&food.ratings.length">
                  <li v-for="(rating,key) in food.ratings" :key="key" v-show="needShow(rating.text,rating.rateType)" class="rating-item" border-1px>
                    <div class="user">
                      <span class="name">{{rating.username}}</span>
                      <img :src="rating.avatar" class="avatar" width="12" height="12">
                    </div>
                    <div class="time">{{rating.rateTime|dateFormat}}</div>
                    <p class="text">
                      <span :class="{'icon-thumb_up':rating.rateType===0,'icon-thumb_down':rating.rateType===1}"></span>{{rating.text}}
                    </p>
                  </li>
                </ul>
                <div class="no-rating" v-show="!food.ratings || !food.ratings.length">暂无评价</div>
              </div>
            </div>
        </div>
    </div>
</transition>
  
</template>

<script>
import BScroll from 'better-scroll';
import cartcontrol from '../cartcontrol/cartcontrol';
import split from '../split/split';
import ratingselect from '../ratingselect/ratingselect';
import formatDate from '../../public/js/date';

const POSITIVE = 0;
const NEGATIVE = 1;
const ALL = 2;

export default {
  props: {
    food: {
      type: Object,
    },
  },
  data() {
    return {
      showFlag: false,
      selectType: ALL,
      onlyContent: true,
      desc: {
        all: '全部',
        positive: '推荐',
        negative: '吐槽',
      },
    };
  },
  components: {
    cartcontrol,
    split,
    ratingselect,
  },
  methods: {
    show() {
      this.showFlag = true;
      this.$nextTick(() => {
        if (!this.foodScroll) {
          this.foodScroll = new BScroll(this.$refs.food, {
            click: true,
          });
        }
        this.foodScroll.refresh();
      });
    },
    hide() {
      this.showFlag = false;
    },
    addFirst() {
      if (!this.food.count) {
        this.$set(this.food, 'count', 1);
      } else {
        this.food.count++;
      }
      this.$emit('cart-add', event.target);
    },
    cartAdd() {
      this.$emit('cart-add', event.target);
    },
    typeChange(type) {
      switch (type) {
        case 0:
          this.selectType = POSITIVE;
          break;
        case 1:
          this.selectType = NEGATIVE;
          break;
        default:
          this.selectType = ALL;
          break;
      }  
      this.$nextTick(() => {
        this.foodScroll.refresh();
      });
    },
    toggleContent(onlyContent) {
      this.onlyContent = onlyContent;
      this.$nextTick(() => {
        this.foodScroll.refresh();
      });
    }, 
    needShow(text, type) {
      if (this.onlyContent && !text) {
        return false;
      }
      if (this.selectType === ALL) {
        return true;
      }
      return type === this.selectType;
    },
  },
  filters: {
    dateFormat(time) {
      let date = new Date(time);
      return formatDate(date, 'yyyy-MM-dd hh:mm');
    },
  },
};
</script>

<style lang="stylus">

@import '../../public/stylus/mixin';

  .food
    position fixed
    top 0
    left 0
    bottom 48px
    z-index 30
    width 100%
    background-color #fff
    &.move-enter-active,&.move-leave-active 
       transition all 0.2s linear
    &.move-enter,&.move-leave-to 
       opacity 0
       transform translate3d(100%,0,0)
    &.move-enter-to,&.move-leave   
       opacity 1
       transform translate3d(0,0,0)
    .image-header 
       position relative
       width 100%
       height 0
       padding-top 100%
       img 
         position absolute
         top 0
         left 0
         width 100%
         height 100%
       .back 
         position absolute
         top 10px
         left 1px
         background-color rgba(7,17,27,0.3)
         border-radius 50%
         .icon-arrow_lift
            display block
            padding 9px
            font-size 20px
            color #fff
    .content
       padding 18px
       position relative
       .title 
         margin-bottom 8px
         font-size 14px
         line-height 14px
         font-weight 700
         color rgb(7,17,27)
       .detail
         margin-bottom 18px
         font-size 0
         line-height 10px
         height 10px 
         .sell-count,.rating 
            font-size 10px
            color rgb(147,153,159)
         .sell-count 
            margin-right 12px
       .price
         font-weight 700
         line-height 24px
         .now
            margin-right 8px
            font-size 14px
            color rgb(240,20,20)
         .old
            text-decoration line-through
            font-size 10px
            color rgb(147,153,159)
       .cartcontrol-wrapper
         position absolute
         right 12px
         bottom 12px
       .buy
         position absolute
         right 18px
         bottom 18px
         z-index 10
         height 24px
         line-height 24px
         padding 0 12px
         font-size 10px
         box-sizing border-box
         border-radius 12px
         color #fff
         background-color rgb(0,160,220)
         &.fade-leave-active 
          transition all 0.2s
         &.fade-leave-to,&.fade-enter 
          opacity 0
    .info 
      padding 18px
      .title
        line-height 14px
        font-size 14px
        margin-bottom 6px
        color rgb(7,17,27)
      .text 
        padding 0 8px
        line-height 24px
        font-size 12px
        color rgb(77,85,93)
    .rating
      padding-top 18px
      .title 
        line-height 14px
        font-size 14px
        margin-left 18px
        color rgb(7,17,27)
      .rating-wrapper
        padding 0 18px
        .rating-item 
          position relative
          padding 16px 0
          border-1px(rgba(7,17,27,0.1))
          .user 
            position absolute
            top 16px
            right 0
            font-size 0
            line-height 12px
            .name 
              margin-right 6px
              font-size 10px
              display inline-block
              vertical-align top
              color rgb(147,153,159)
            .avatar
              border-radius 50%
          .time 
            margin-bottom 6px
            line-height 12px
            font-size 10px
            color rgb(147,153,159)
          .text
            line-height 16px
            font-size 12px
            color rgb(7,17,27)
            .icon-thumb_up,icon-thumb_down
              margin-right 4px
              line-height 16px
              font-size 12px
            .icon-thumb_up
              color rgb(0,160,220)
            .icon-thumb_down
              color rgb(147,153,159)
        .no-rating 
          padding 16px 0
          font-size 12px
          color rgb(147,153,159)   
</style>


