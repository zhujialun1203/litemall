<template>
  <div class="tab_class">
    <div class="tal_class_searchBox">
      <van-search placeholder="点击前往搜索" />
      <div class="tal_class_searchMask" @click="$router.push({ name: 'search' })"></div>
    </div>
    <!-- <div class="van-coupon-item"
           v-for="(coupon,index) in shopInfos.couponList"
           :key="index"
           @click="getCoupon(coupon.id)"> -->
    <div class="centered-text" style="color: black;">抢购</div>
    <div class="van-coupon-item__content">
      <span class="centered-text">进入抢购</span>
    </div>
    <div style="color: #00bfff;  font-weight: bold; padding-left: 10px;">公告通知</div>
    <div class="van-coupon-item__content2">
      <span class="centered-text">XXXXX</span>
     
    </div>

    <div class="van-coupon-item__content3">
      <span class="centered-text">您已经抢了单      共计金额元</span>
      <span class="centered-text">委卖后收益是：XX元</span>
      <span class="centered-text">上架手续费：XX元</span>
    </div>


    <!-- </div> -->

    <!-- <div class="class_tree clearfix">
    <ul class="class_tree_nav">
      <li
        v-for="(item, index) in categoryList"
        :key="index"
        :class="{active_nav: currentCategory.id == item.id}"
        @click="changeCatalog(item.id)"
      >{{item.name}}</li>
    </ul>
    <div class="class_tree_content">
      <div class="class_tree_all">
        <img style="width:250px" v-lazy="currentCategory.picUrl">
      </div>
      <div class="box">
        <span>{{currentCategory.desc}}</span>
      </div>
      <div class="class_tree_items_wrap clearfix">
        <div @click="toItemList(item.id)" :key="i" v-for="(item, i) in currentSubCategoryList">
          <div class="class_tree_item_img">
            <img :src="item.picUrl" :alt="item.name">
          </div>
          <div class="class_tree_item_name">{{item.name}}</div>
        </div>
      </div>
    </div>
  </div> -->

  </div>
</template>

<script>
import { catalogList, catalogCurrent } from '@/api/api';

import { Search } from 'vant';

export default {
  data() {
    return {
      categoryList: [],
      shopInfos: [],
      currentCategory: {},
      currentSubCategoryList: []
    };
  },

  created() {
    this.initData();
  },

  methods: {
    initData() {
      catalogList().then(res => {
        let data = res.data.data;
        this.categoryList = data.categoryList;
        this.currentCategory = res.data.data.currentCategory;
        this.currentSubCategoryList = data.currentSubCategory;
      });
      this.shopInfos = {
        couponList: [{
          "id": 1,
          "name": "限时满减券",
          "desc": "全场通用",
          "tag": "无限制",
          "discount": 5,
          "min": 99,
          "days": 10
        },
        {
          "id": 2,
          "name": "限时满减券",
          "desc": "全场通用",
          "tag": "无限制",
          "discount": 5,
          "min": 99,
          "days": 10
        },
        {
          "id": 2,
          "name": "限时满减券",
          "desc": "全场通用",
          "tag": "无限制",
          "discount": 5,
          "min": 99,
          "days": 10
        }
        ]
      }
    },
    changeCatalog(id) {
      catalogCurrent({ id: id }).then(res => {
        let data = res.data.data;
        this.currentCategory = data.currentCategory;
        this.currentSubCategoryList = data.currentSubCategory;
      });
    },
    toItemList(id) {
      this.$router.push({
        name: 'category',
        query: { keyword: '', itemClass: id }
      });
    }
  },
  components: {
    [Search.name]: Search
  }
};
</script>


<style lang="scss" scoped>
@import '../../assets/scss/mixin';

.centered-text {
  text-align: center;
  /* 文字内容居中 */
  font-weight: bold;
  /* 加粗 */
  color: white;
}


.van-coupon-item__content {
  display: -webkit-box;
  display: -ms-flexbox;
  display: flex;
  flex-direction: column;
  justify-content: center;
  height: 150px;
  padding: 24px 10px 10px 15px;
  -webkit-box-sizing: border-box;
  box-sizing: border-box;

  margin-top: 10px;
  margin-left: 10px;
  margin-right: 10px;
  border-radius: 10px;

  background-color: #faf0e6;
}

.van-coupon-item__content2 {
  display: -webkit-box;
  display: -ms-flexbox;
  display: flex;
  flex-direction: column;
  justify-content: center;
  height: 150px;
  padding: 24px 10px 10px 15px;
  -webkit-box-sizing: border-box;
  box-sizing: border-box;
  
  margin-top: 10px;
  margin-left: 10px;
  margin-right: 10px;
  border-radius: 10px;


  background-image: url('../../../public/resource/7582358408a9512592068a4dc25a92a7.jpg');
  background-size: cover;
  background-position: center;
  background-repeat: no-repeat;
  text-align: center;
}

.van-coupon-item__content3 {
  display: -webkit-box;
  display: -ms-flexbox;
  display: flex;
  flex-direction: column;
  justify-content: center;
  height: 150px;
  padding: 24px 0 0 15px;
  -webkit-box-sizing: border-box;
  box-sizing: border-box;

  margin-top: 10px;
  margin-left: 10px;
  margin-right: 10px;
  border-radius: 10px;

  background-color: #ff450f;
}


.tab_class {
  overflow: hidden;
  background-color: #fff;
}

.height-fix {
  padding-bottom: 42px;
}

.tal_class_searchBox {
  position: relative;
}

.tal_class_searchMask {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 9;
}


.box {
  width: 250px;
  height: 20px;
  text-align: center;
  font-family: PingFangSC-Light, helvetica, 'Heiti SC';
  font-size: 13px;
  position: absolute;
  top: 95px;
}

.box span {
  line-height: 20px;
}

.class_tree {
  position: relative;
  background-color: #fff;
  overflow-x: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  overflow: hidden;
  height: 100%;
  box-sizing: border-box;
}

.class_tree_nav {
  float: left;
  width: 100px;
  height: 100%;
  background-color: #fff;
  overflow: scroll;

  >li {
    @include one-border;
    height: 40px;
    line-height: 40px;
    text-align: center;
    border-left: 2px solid $bg-color;
  }

  >li.active_nav {
    background-color: #fff;
    border-left: 2px solid $red;
    color: $red;
  }
}

.class_tree_content {
  margin-left: 100px;
  height: 100%;
  overflow-x: hidden;
  overflow-y: scroll;

  .class_tree_all {
    text-align: right;
    padding-right: 10px;
    height: 40px;
    line-height: 40px;
    color: $font-color-gray;
    font-size: $font-size-small;
  }

  .van-icon-arrow {
    font-size: $font-size-small;
  }

  .class_tree_items_wrap {
    padding: 10px 20px;
    margin-right: -3%;
    margin-top: 70px;
    text-align: center;

    >div {
      float: left;
      padding-right: 3%;
      box-sizing: border-box;
      width: 33.333%;
      margin-bottom: 20px;
    }

    img {
      max-width: 100%;
    }

    .class_tree_item_img {
      display: inline-block;
      max-width: 100%;
      width: 70px;
      height: 70px;
    }

    .class_tree_item_name {
      overflow: hidden;
      text-overflow: ellipsis;
      white-space: nowrap;
    }

  }

}
</style>
