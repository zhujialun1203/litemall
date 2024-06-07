<template>
  <div class="tab_class">
    <div class="tal_class_searchBox">
      <van-search placeholder="点击前往搜索" />
      <div class="tal_class_searchMask" @click="$router.push({ name: 'search' })"></div>
    </div>

    <div class="centered-text" style="color: black;"  >抢购</div>
    <div class="van-coupon-item__content">
      <span :disabled="isButtonDisabled" @click="toItemList(1005000)" class="centered-text">{{remainingTimeText}}</span>
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
      isButtonDisabled: true, // 默认按钮不可用
      remainingTime: 0, // 剩余时间，单位分钟
      timer: null, // 定时器引用

      categoryList: [],
      shopInfos: [],
      currentCategory: {},
      currentSubCategoryList: [],

      startDate:new Date("10:00"),
      endDate:new Date("11:00")
    };
  },
  computed: {
    // 计算属性，判断当前时间是否在可用时间段内
    isWithinOperatingHours() {
      const now = new Date();
      return this.startDate<= now <  this.endDate;
    },
    remainingTimeText() {
      if (this.remainingTime > 0) {
        const minutes = Math.floor(this.remainingTime / 60);
        const seconds = this.remainingTime % 60;
        return `剩余${minutes}:${seconds < 10 ? '0' + seconds : seconds}秒可点击`;
      } else if (this.isButtonDisabled) {
        return "不可点击";
      } else {
        return "点击";
      }
    },
  },

  created() {
    this.initData();
  },
  mounted() {
 // 页面加载时立即检查按钮状态
    this.updateButtonStatus();
    // 每秒检查一次，以应对时间变化
    setInterval(this.updateButtonStatus, 1000);
  },
  beforeDestroy() {
    if (this.timer) {
      clearInterval(this.timer);
  }},

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
      if (this.isButtonDisabled) {
        return; // 如果按钮处于禁用状态，则不执行点击逻辑
      }
      this.$router.push({
        name: 'all',
        query: { keyword: '', itemClass: id }
      });
    },

    updateButtonStatus() {
      if (this.isWithinOperatingHours) {
        // 如果在可用时间段内
        this.isButtonDisabled = false;
        if (this.timer) {
          clearInterval(this.timer); // 清除可能存在的倒计时
        }
      } else {
         // 计算距离下一个可点击时间段的剩余时间（以秒为单位）
        const now = new Date();
        let nextStartHour = (now.getHours() >= this.endDate.getHours && now.getMinutes() > this.endDate.minutes) ? this.startDate.getHours : now.getHours();
        const nextStartTime = new Date(now.getFullYear(), now.getMonth(), now.getDate(), nextStartHour, this.startDate.getMinutes, 0, 0);
        if (nextStartTime < now) {
          // 如果当前时间已经超过今天的目标时间段，设置为明天的9:50
          nextStartTime.setDate(nextStartTime.getDate() + 1);
        }
        const timeUntilNextSlot = nextStartTime - now; // 距离下一个可用时间的毫秒数
        this.remainingTime = Math.ceil(timeUntilNextSlot / 1000); // 转换为秒

        if (this.remainingTime <= 60) {
          // 开始倒计时，这里以1分钟为例，根据需要调整
          this.startCountdown();
        } else {
          this.isButtonDisabled = true;
        }
      }
    },
    startCountdown() {
       this.timer = setInterval(() => {
        if (this.remainingTime <= 0) {
          clearInterval(this.timer);
          this.updateButtonStatus(); // 刷新状态，准备进入下一个可用时间段
        } else {
          this.remainingTime--;
        }
      }, 1000); // 每秒更新一次
    },
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
  color: black;
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


  background-image: url('');
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
