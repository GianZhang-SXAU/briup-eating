<template>
  <div class="orderDetail">
    <div class="header">
      <!--      导航-->
      <van-nav-bar
          fixed
          left-arrow
          title="订单详情"
          @click-left="onClickLeft"
          @click-right="onClickRight"
      >
        <template #right>
          <div v-if="order.status === '待支付'">
            <span>支付</span>
          </div>
        </template>
      </van-nav-bar>
    </div>
    <div class="container">
      <div class="status">
        <van-steps :active="active" active-icon="success" active-color="#ee0a24">
          <van-step>选购菜品</van-step>
          <van-step>待支付</van-step>
          <van-step>待派送</van-step>
          <van-step>已完成</van-step>
        </van-steps>
        <van-progress color="#ee0a24" :percentage="orderProcess" />
      </div>
      <div class="info">
        <!--        收获地址及其他信息-->
        <div class="address">
          <span class="title"> 地址：</span>
          <span>
            {{ order.address.province }}
            {{ order.address.city }}
            {{ order.address.area }}
          </span>
        </div>
        <div class="name">
          <span class="title"> 姓名：</span>
          <span>{{ order.address.realname }}</span>
        </div>
        <div class="tel">
          <span class="title"> 电话：</span>
          <span>{{ order.address.telephone }}</span>
        </div>
        <div class="message">
          <span class="title"> 备注：</span>
          <span>{{ order.buyerMessage }}</span>
        </div>
      </div>
      <div class="orderLines">
        <!--        订单菜品-->
        <van-card
            v-for="item in order.orderLines"
            :key="item.id"
            :num="item.num"
            :price="item.price.toFixed(2)"
            :title="item.name"
            :thumb="item.photo"
        >
          <template #desc>
            <!-- 描述信息最多显示一行，超出部分省略 -->
            <div class="van-ellipsis">
              {{ item.product.description }}
            </div>
          </template>
        </van-card>
      </div>
    </div>
  </div>
</template>

<script>
import {get} from "@/http/axios";
import Vue from 'vue';
import { Toast } from 'vant';

Vue.use(Toast);
export default {
  data() {
    return {
      order: JSON.parse(this.$route.query.item),
      active : '1',
      orderProcess : 25
    }
  },
  methods: {
    onClickLeft() {
      this.$router.go(-1)
    },
    // 支付
    async onClickRight(){
     let res = await get('/app/order/pay',{id:this.order.id})
       if (res.data.status === 200){
         Toast.success(res.data.message)
         this.$router.push('/manager/order')
       }else {
         Toast.fail(res.data.message)
       }
      console.log(res)
    },
    //转变订单状态
    turnStatus(){
      if (this.order.status == '待支付'){
        this.active = 1
        this.orderProcess = 50
      } else if (this.order.status == '待派送'){
        this.active = 2
        this.orderProcess = 75
      } else {
        this.active = 3
        this.orderProcess = 100
      }
    }
  },
  created() {
    console.log(this.$route.query)
    this.turnStatus()
  }
}
</script>
<style lang="less" scoped>
.header {
  .van-nav-bar {
    background-color: #ee0a24;

    ::v-deep .van-icon {
      color: white;
    }

    ::v-deep .van-nav-bar__title {
      color: white;
    }
    ::v-deep .van-nav-bar__right {
      color: white;
    }
  }
}

.container {
  //padding-top: 46px;
  padding: 46px 10px 10px;
  box-sizing: border-box;
  height: 100vh;
  background-color: #f2f3f5;
  overflow-y: scroll;

  .status {
    margin-top: 10px;
  }

  .info {
    margin: 10px 0;
    background-color: #ffffff;
    border-radius: 10px;
    padding: 10px;
    display: flex;
    flex-wrap: wrap;
    font-size: 14px;
    color: #898989;
    box-shadow: 0 0 6px 0 #ccc;
  }

  .address, .message {
    width: 100%;
  }

  .name, .tel {
    width: 50%;
    margin: 5px 0;
  }

  .title {
    color: #666666;
  }

  .orderLines {
    background-color: #ffffff;
    border-radius: 10px;
    padding: 10px;
    .van-card__title {
      font-size: 14px;
      line-height: 25px;
      font-weight: 600;
      box-shadow: 0 0 1px 0 #ccc;
    }
  }
}
</style>