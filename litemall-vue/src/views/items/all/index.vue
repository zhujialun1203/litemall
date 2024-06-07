<template>

    <van-list
      v-model:loading="isLoading"
      :finished="isFinished"
      finished-text="没有更多数据了"
      @load="onLoadMore"
    >
      <div class="waterfall-container">
        <div 
          class="waterfall-column" 
          v-for="(column, columnIndex) in columns"
          :key="columnIndex"
        >
          <van-card
            v-for="item in column"
            :key="item.id"
            :desc="item.brief"
            :title="item.name"
            :thumb="item.picUrl"
            :price="item.retailPrice"
            @click="itemClick(item.id)"
          />
        </div>
      </div>
    </van-list>

</template>




<script>
import { goodsList } from '@/api/api';
import { Card, List } from 'vant';
import scrollFixed from '@/mixin/scroll-fixed';

export default {
  mixins: [scrollFixed],

  data() {
    return {
      list: [], // 初始商品列表数据
      columns: [[]], // 用于存放双列的卡片数据
      isLoading: false, // 是否正在加载更多
      isFinished: false, // 是否已加载完所有数据
      isPullingDown: false, // 是否正在下拉刷新
      pageNo: 1, // 当前页码
      pageSize: 10, // 每次加载的数据量
    };
  },

  created() {
    this.init();
  },

  methods: {
    init() {
      this.list = [];
    },
    onRefresh() {
      this.isPullingDown = true;
      this.pageNo = 1;
      this.list = []; // 清空现有数据准备重新加载
      this.columns = [[]]; // 重置列
      this.onLoadMore().then(() => {
        this.isPullingDown = false;
      });
    },
    onLoadMore() {
      this.isLoading = true;
      // 假设getGoodList是一个异步函数，用于获取商品列表数据
      return this.getGoodList(this.pageNo, this.pageSize)
        .then((newData) => {
          var data=newData.data.data.list;
          if (data.length) {
            this.list.push(...newData.data.data.list);
            this.redistributeCards();
            if (data.length < this.pageSize) {
              this.isFinished = true;
            }
            this.pageNo++;
          } else {
            this.isFinished = true;
          }
          this.isLoading = false;
        })
        .catch((error) => {
          console.error('加载失败:', error);
          this.isLoading = false;
        });
    },
    redistributeCards() {
      this.columns = [[]];
      this.list.forEach((item, index) => {
        const columnIndex = index % 2; // 根据索引分配到不同列
        if (!this.columns[columnIndex]) {
          this.columns[columnIndex] = [];
        }
        this.columns[columnIndex].push(item);
      });
    },
    getGoodList(page,size) {
      return goodsList({page,size})
    },
    itemClick(id) {
      this.$router.push(`/items/detail/${id}`);
    }
  },

  components: {
    [List.name]: List,
    [Card.name]: Card
  }
};
</script>

<style lang="scss" scoped>
.waterfall-container {
  display: flex;
  flex-wrap: wrap;
}

.waterfall-column {
  width: calc(50% - 10px); /* 调整宽度以适应双列，并考虑列间间距 */
  margin-right: 10px;
}

.waterfall-column:last-child {
  margin-right: 0;
}

.goods_hot {
  padding: 20px;
  .banner {
    height: 250px;
    background-image: url('http://yanxuan.nosdn.127.net/8976116db321744084774643a933c5ce.png');
    background-size: cover;
    margin-bottom: 20px;
    .title {
      text-align: center;
      line-height: 200px;
      color: #ffffff;
      font-size: 40px;
    }
  }
}
</style>