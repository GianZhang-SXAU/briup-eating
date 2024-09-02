<template>
  <div class="home">
    <!--      搜索-->
    <div class="header">
      <van-search
          v-model="value"
          show-action
          background="#ee0a24"
          placeholder="请输入菜品关键词"
          shape="round"
          @clear
      >
        <template #action>
          <div @click="onSearch" class="search">搜索</div>
        </template>
      </van-search>
    </div>
    <!--      轮播图-->
    <div class="images">
      <van-swipe :autoplay="3000">
        <van-swipe-item v-for="(image, index) in images" :key="index">
          <img v-lazy="image.url" height="180px" width="100%"/>
        </van-swipe-item>
      </van-swipe>
    </div>
    <!--      宫格-->
    <div class="container">
      <van-grid :column-num="3" :border="0">
        <van-grid-item :to="{path: '/product',query: {index}}" v-for="(item,index) in categorys" :key="item.id" :icon="item.icon" :text="item.name"/>
      </van-grid>
    </div>
    <!--      搜索结果-->
    <div class="product">
      <van-card v-for="item in result" :key="item.id"
          :price="item.price.toFixed(2)"
          :desc="item.description"
          :title="item.name"
          :thumb="item.photo"
      >
<!--        菜品分类的标签-->
        <template #tags>
          <van-tag plain type="danger">{{ item.category.name }}</van-tag>
        </template>
        <template #num>
          <div>
            {{item.unit}}
          </div>
        </template>
      </van-card>
    </div>
  </div>
</template>

<script>
// 引入一个get请求
import {get} from '@/http/axios'
import Vue from 'vue';
import {Lazyload} from 'vant';


Vue.use(Lazyload);
export default {
  computed: {

  },
  data() {
    return {
      //搜索的变量
      value: '',
      images: [
        'https://img01.yzcdn.cn/vant/apple-1.jpg',
        'https://img01.yzcdn.cn/vant/apple-2.jpg',
      ],
      //菜品分类
      categorys:[],
      // 是否显示边框
      show : false,
      // 搜索结果数组
      result:[],
    }

  },
  watch:{
    // 监听搜索框，值为空清空搜索结果
    value(newValue){
      if (newValue == ''){
        this.result = []
      }
    }
  },
  methods: {

    //搜索方法
    async onSearch() {
        let param = {
          page:1,
          pageSize:10,
          name: this.value
        }
        let res = await  get('/product/pageQuery',param)
        console.log(res.data.data.list)
        this.result = res.data.data.list

    },
    // 获取轮播图
    async getImages() {
      let res = await get('/carousel/findAll')
      // console.log(res)
      this.images = res.data.data
    },
    //获取商品树
    async getAll(){
      let {data} = await get('/app/product/queryCategoryWithProducts')
      console.log(data)
       this.categorys = data.data
    }
  },
  created() {
    this.getImages()
    this.getAll()
  }
}
</script>
<style lang="less" scoped>
.home{
  .header{
    position: fixed;
    top: 0;
    width: 100%;
    z-index: 10;
    .search{
      color: #fff;
    }
  }
  // 轮播图
  .images{
    padding-top: 54px;
  }
  .product{
    .van-card:last-child{
      margin-bottom: 50px;
    }
  }
}
</style>