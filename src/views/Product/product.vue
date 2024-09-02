<template>
  <div class="product">
    <div class="header">
      <!--      导航-->
      <van-nav-bar
          fixed
          left-arrow
          title="菜品分类"
          @click-left="onClickLeft"
      />
    </div>
    <div class="container">
      <!--      菜品内容：分类和具体菜品-->
      <div class="left">
        <van-sidebar v-model="activeKey" @change="onChange">
          <van-sidebar-item v-for="item in list" :key="item.id" :title="item.name"/>
        </van-sidebar>
      </div>
      <div class="right">
        <van-card
            v-for="item in products"
            :key="item.id"
            :price="item.price.toFixed(2)"
            :thumb="item.photo"
            :title="item.name"
        >
          <template #desc>
            <!--            描述信息，最多显示一行，超出省略-->
            <div class="van-ellipsis">{{ item.description }}</div>
          </template>
          <template #num>
            <div>
              <van-stepper v-model="item.num" button-size="22" default-value="0" disable-input
                           min="0" theme="round" @change="changeShopCart(item)"/>
            </div>
          </template>
        </van-card>

      </div>
      <div class="bottom">
        <div class="total" @click="showOverlay">总计：￥ {{ total.toFixed(2) }}</div>
        <div class="btn" @click="toOrder">去下单</div>
      </div>
      <!--      遮罩层-->

      <van-overlay :show="show" @click="show = false">
        <div class="wrapper">
          <div class="block" @click.stop>
<!--            清空购物车-->
            <div class="clear" @click="clear">
              清空购物车<van-icon name="delete-o" />

            </div>
<!--            购物车里的菜品-->
            <van-card
                v-for="value of orderLines.values()"
                :key="value.id"
                :price="value.price.toFixed(2)"
                :thumb="value.photo"
                :title="value.name"
            >
              <template #desc>
                <!--            描述信息，最多显示一行，超出省略-->
                <div class="van-ellipsis">{{ value.description }}</div>
              </template>
              <template #num>
                <div>
                  <van-stepper v-model="value.num" button-size="22" disable-input
                               min="1" theme="round" @change="changeShopCart(value)"/>
                </div>
              </template>
              <template #footer>
                <van-icon name="clear" @click="delPro(value)" color="#bbb" size="1rem"/>
              </template>
            </van-card>
          </div>
        </div>
      </van-overlay>
    </div>
    <!--    显示总价格和下单按钮-->

  </div>
</template>

<script>
import {mapGetters, mapMutations, mapState} from 'vuex'
import {get} from '@/http/axios'
import Vue from 'vue';
import { Toast } from 'vant';

Vue.use(Toast);
export default {
  data() {
    return {
      // 侧边导航发当前索引
      activeKey: this.$route.query.index,
      //   菜品数组:菜品分类与具体的菜品
      list: [],
      //   具体菜品数组
      products: [],
      // 是否显示遮罩层
      show: false
    }
  },
  computed: {
    ...mapState('shopcart', ['orderLines']),
    ...mapGetters('shopcart', ['total'])
  },
  watch:{
    //价格为0 关闭遮罩
    total(newValue){
      if (newValue == 0){
        this.show == false
      }
    }
  },
  methods: {
    ...mapMutations('shopcart', ['addShopcart','clearShopcart']),

    onClickLeft() {
      this.$router.go(-1)
    },
    // 根据orderLines显示菜品数量
    changeProducts(){
      // 将购物车的菜品数量反应到步进器
      this.orderLines.forEach((value, key) => {
        // value.productsCategoryId
        this.list.forEach(item => {
          if (item.id == value.productCategoryId) {
            item.products.forEach(product => {
              if (product.id == key) {
                product.num = value.num
              }
            })
          }
        })
      })
      this.products = this.list[this.activeKey].products
    },
    //获取商品树
    async getAll() {
      let {data} = await get('/app/product/queryCategoryWithProducts')
      console.log(data.data)
      this.list = data.data
      // 将购物车的菜品数量反应到步进器
      this.orderLines.forEach((value, key) => {
        // value.productsCategoryId
        this.list.forEach(item => {
          if (item.id == value.productCategoryId) {
            item.products.forEach(product => {
              if (product.id == key) {
                product.num = value.num
              }
            })
          }
        })
      })
      this.products = this.list[this.activeKey].products
    },
    //   侧边导航栏改变事件
    onChange(index) {
      this.products = this.list[index].products
    },
    // 步进器改变事件方法
    changeShopCart(item) {
      item.productId = item.id
      this.addShopcart(item)
      this.getAll()
    },
    //清空购物车
    clear(){
      this.clearShopcart()
      this.show = false
      this.getAll()
      Toast.success("购物车已成功清空")
    },
    //点击价格显示遮罩层
    showOverlay(){
      if (this.total !== 0){
        // 总价格不等于0时，显示遮罩层
        this.show = true
      }
    },
    //跳转到下单页面
    toOrder(){
      this.$router.push({
        path: '/orderConfirm'
          })
    },
    delPro(item){
      item.productId = item.id
      item.num = 0
      this.addShopcart(item)
      this.getAll()
    }
  },
  created() {
    this.getAll()
    // console.log(this.$route)
  }
}
</script>
<style lang="less" scoped>
.product {
  .header {
    .van-nav-bar {
      background-color: #ee0a24;

      ::v-deep .van-icon {
        color: white;
      }

      ::v-deep .van-nav-bar__title {
        color: white;
      }
    }
  }

  .container {
    width: 100vw;
    height: 100vh;
    overflow-y: scroll;
    padding-top: 46px;
    display: flex;
    background-color: #f2f3f4;

    .left {
      position: fixed;
    }

    .right {
      flex: 1;
      height: ~"calc(100vh - 66px)";
      overflow-y: scroll;
      margin-left: 80px;
      width: ~"calc(100vw - 80px)";

      .van-card__title {
        font-size: 14px;
        line-height: 25px;
        font-weight: 600;
      }

      .van-ellipsis {
        color: darkgrey;
      }
    }

    .bottom {
      @height: 40px;

      position: fixed;
      bottom: 0;
      line-height: @height;
      height: @height;
      margin: 10px;
      width: ~"calc(100vw - 20px)";
      color: white;
      display: flex;
      z-index: 10;

      .total {
        font-size: 20px;
        flex: 1;
        background-color: black;
        padding-left: 20px;
        border-top-left-radius: 20px;
        border-bottom-left-radius: 20px;
      }

      .btn {
        text-align: center;
        font-size: 18px;
        width: 80px;
        background-color: #ee0a24;
        border-top-right-radius: 20px;
        border-bottom-right-radius: 20px;
      }
    }

    //遮罩层
    .wrapper {
      height: 100%;
      display: flex;
      align-items: flex-end;
    }

    .block {
      //padding-top: 10px;
      border-top-left-radius: 20px;
      border-top-right-radius: 20px;
      width: 100vw;
      max-height: 60vh;
      overflow-y: scroll;
      background-color: #fff;

    }
    .clear{
      border-top-left-radius: 20px;
      border-top-right-radius: 20px;
      padding: 5px 0;
      font-size: 14px;
      color: #d3d5d7;
      height: 20px;
      display: flex;
      justify-content: center;
      align-items: flex-end;
      position: sticky;
      top: 0;
      z-index: 10;
      width: 100%;
      background-color: #ffffff;
      &:hover{
        background-color: rgba(239,10,36,0.5);
      }
    }
    .van-card:nth-child(2){
       //margin-top: 10px;
    }
    .van-card:last-child{
      margin-bottom: 60px;
    }
    .van-card__footer{
      float: right;
      margin-top: -100px;
    }
  }
}
</style>