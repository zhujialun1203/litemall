<template>
  <div class="payment">
    <div class="time_down payment_group">
      请在
      <span class="red">半小时内</span>
      完成付款，否则系统自动取消订单
    </div>

    <van-cell-group class="payment_group">
      <van-cell title="订单编号" :value="order.orderInfo.orderSn" />
      <van-cell title="实付金额">
        <span class="red">{{ order.orderInfo.actualPrice * 100 | yuan }}</span>
      </van-cell>
    </van-cell-group>

    <div class="pay_way_group">
      <div class="pay_way_title">选择支付方式</div>
      <van-radio-group v-model="payWay">
        <van-cell-group>
          <van-cell>
            <template slot="title">
              <img src="../../../assets/images/ali_pay.png" alt="支付宝" width="82" height="29">
            </template>
            <van-radio name="ali">
            </van-radio>

          </van-cell>
          <van-cell>
            <template slot="title">
              <img src="../../../assets/images/wx_pay.png" alt="微信支付" width="113" height="23">
            </template>
            <van-radio name="wx" />
          </van-cell>
          <van-cell>
            <template slot="title">
              <img src="../../../assets/images/wx_pay.png" alt="其他支付" width="113" height="23">
            </template>
            <van-radio name="other" />
          </van-cell>
        </van-cell-group>
      </van-radio-group>
      <img src="https://pic.ntimg.cn/file/20200513/27111845_133921058002_2.jpg" stalt="二维码" class="center-image">
      <van-uploader :afterRead="avatarAfterRead" class="pay_upload_pos">
        <div class="pay_upload">
          <img :src="avatar + '?x-oss-process=image/resize,m_fill,h_50,w_50'" alt="支付凭证" v-if="avatar">
          <van-icon name="camera_full" v-else></van-icon>
        </div>
      </van-uploader>
    </div>


    <van-button class="pay_submit" @click="pay" type="primary" bottomAction>确认支付</van-button>
  </div>
</template>

<script>
import { Uploader, Radio, RadioGroup, Dialog } from 'vant';
import { orderDetail, orderPrepay, orderH5pay,upload} from '@/api/api';
import _ from 'lodash';
import { getLocalStorage, setLocalStorage } from '@/utils/local-storage';
import axios from 'axios';

export default {
  name: 'payment',


  data() {
    return {
      avatar: '',
      payWay: 'wx',
      file:null,
      order: {
        orderInfo: {},
        orderGoods: []
      },
      orderId: 0
    };
  },
  created() {
    if (_.has(this.$route.params, 'orderId')) {
      this.orderId = this.$route.params.orderId;
      this.getOrder(this.orderId);
    }
  },
  methods: {
    getOrder(orderId) {
      orderDetail({ orderId: orderId }).then(res => {
        this.order = res.data.data;
      });
    },
    avatarAfterRead(file) {
      const formData = new FormData();
      formData.append('file', file.content); // 将文件附加到 FormData 对象
      upload(formData).then(response => {
        console.log('文件上传成功:', response);
        // 如果需要，可以在这里更新 avatar 的值，例如使用上传后的文件 URL
        this.avatar = response.data.url;
      })
      .catch(error => {
        console.error('文件上传失败:', error);
      });
      console.log(file);
    },
    pay() {
      this.$router.push('/user/order/list/1')
      // Dialog.alert({
      //   message: '你选择了' + (this.payWay === 'wx' ? '微信支付' : '支付宝支付')
      // }).then(() => {
      //   if (this.payWay === 'wx') {
      //     let ua = navigator.userAgent.toLowerCase();
      //     let isWeixin = ua.indexOf('micromessenger') != -1;
      //     if (isWeixin) {
      //       orderPrepay({ orderId: this.orderId })
      //         .then(res => {
      //           let data = res.data.data;
      //           let prepay_data = JSON.stringify({
      //             appId: data.appId,
      //             timeStamp: data.timeStamp,
      //             nonceStr: data.nonceStr,
      //             package: data.packageValue,
      //             signType: 'MD5',
      //             paySign: data.paySign
      //           });
      //           setLocalStorage({ prepay_data: prepay_data });

      //           if (typeof WeixinJSBridge == 'undefined') {
      //             if (document.addEventListener) {
      //               document.addEventListener(
      //                 'WeixinJSBridgeReady',
      //                 this.onBridgeReady,
      //                 false
      //               );
      //             } else if (document.attachEvent) {
      //               document.attachEvent(
      //                 'WeixinJSBridgeReady',
      //                 this.onBridgeReady
      //               );
      //               document.attachEvent(
      //                 'onWeixinJSBridgeReady',
      //                 this.onBridgeReady
      //               );
      //             }
      //           } else {
      //             this.onBridgeReady();
      //           }
      //         })
      //         .catch(err => {
      //           Dialog.alert({ message: err.data.errmsg });
      //           that.$router.replace({
      //             name: 'paymentStatus',
      //             params: {
      //               status: 'failed'
      //             }
      //           });
      //         });
      //     } else {
      //       orderH5pay({ orderId: this.orderId })
      //         .then(res => {
      //           let data = res.data.data;
      //           window.location.replace(
      //             data.mwebUrl +
      //             '&redirect_url=' +
      //             encodeURIComponent(
      //               window.location.origin +
      //               '/#/?orderId=' +
      //               this.orderId +
      //               '&tip=yes'
      //             )
      //           );
      //         })
      //         .catch(err => {
      //           Dialog.alert({ message: err.data.errmsg });
      //         });
      //     }
      //   } else {
      //     //todo : alipay
      //   }
      // });
    },
    onBridgeReady() {
      let that = this;
      let data = getLocalStorage('prepay_data');
      // eslint-disable-next-line no-undef
      WeixinJSBridge.invoke(
        'getBrandWCPayRequest',
        JSON.parse(data.prepay_data),
        function (res) {
          if (res.err_msg == 'get_brand_wcpay_request:ok') {
            that.$router.replace({
              name: 'paymentStatus',
              params: {
                status: 'success'
              }
            });
          } else if (res.err_msg == 'get_brand_wcpay_request:cancel') {
            that.$router.replace({
              name: 'paymentStatus',
              params: {
                status: 'cancel'
              }
            });
          } else {
            that.$router.replace({
              name: 'paymentStatus',
              params: {
                status: 'failed'
              }
            });
          }
        }
      );
    }
  },

  components: {
    [Radio.name]: Radio,
    [RadioGroup.name]: RadioGroup,
    [Dialog.name]: Dialog,
    [Uploader.name]: Uploader,
  }
};
</script>

<style lang="scss" scoped>
.payment_group {
  margin-bottom: 10px;
}

.time_down {
  background-color: #fffeec;
  padding: 10px 15px;
}

.pay_submit {
  position: fixed;
  bottom: 0;
  width: 100%;
}

.pay_way_group img {
  vertical-align: middle;
}

.center-image {
  width: 75%;
  height: 100%;
  padding-left: 25%;
  padding-bottom: 10px;
  padding-top: 10px;
}

// .pay_way_group {
//   display: flex;
//   justify-content: center;
//   align-items: center;
//   height: 200px; /* 根据需要调整 */
//   width: 100%;
// }
.pay_way_title {
  width: 100%;
  padding: 15px;
  background-color: #fff;
}

.pay_upload_pos {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 200px;
  /* 根据需要调整 */
}

.pay_upload {
  position: relative;
  width: 200px;
  height: 200px;
  border: 1px solid $border-color;

  img {
    max-width: 100%;
    max-height: 100%;
  }

  i {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    font-size: 20px;
    color: $border-color;
  }
}
</style>
