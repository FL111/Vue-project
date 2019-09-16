<template>
    <div>
        <header class="home-header wrap" :class="{'active' : headerActive}">
            <div class="header-search">
                <i class="iconfont icon-search"></i>
                <router-link tag="span" class="search-title" to="./search">请输入要搜索的商品</router-link>
            </div>
            <router-link tag="span" to="./login" v-if="!isLogin">登录</router-link>
            <router-link tag="span" to="./user" v-else>
                <i class="iconfont icon-iconyonghu"></i>
            </router-link>
        </header>

        <nav-bar></nav-bar>
      <!--<HR/>-->
        <slider :imgUrl="headList"></slider>
        <div class="category-list">
            <div v-for="item in categoryList" @click="selectCategory(item.id)">
                <img :src="item.img">
                <span>{{item.name}}</span>
            </div>
        </div>
        <HR style="display: block; margin-bottom: 10px"/>
        <img src="../../assets/category/new.png" style="margin-left: 45%">
      <div class="product-list">
          <!--<refresh :on-refresh="onRefresh" :on-infinite="onInfinite">-->
            <div>
              <div class="product-item" v-for="(item,index) in productList" @click="productDetail(index)">
                <img :src="'http://img.cdn.imbession.top/'+item.mainImage" alt=""/>
                <div class="product-info">
                  <p class="name">{{item.name}}</p>
                  <p class="subtitle">{{item.subtitle}}</p>
                  <span class="price">￥ {{item.price}}</span>
                </div>
              </div>
            </div>
          <!--</refresh>-->
      </div>
    </div>
</template>

<script>
    import refresh from '../../components/common/refresh'
    import mHeader from 'components/mHeader'
    import navBar from 'components/navBar'
    import slider from 'components/common/slider'
    import {mapMutations} from 'vuex'

    import {homeData, checkLogin, headData} from "../../service/getData";
    import {dedupeObject, getStore, setStore} from "../../common/js/util";
    import {productListKeyword, productListCategoryId} from "../../service/getData";

    export default {
        data() {
            return {
                headList: [],
                categoryList: [],
                productList: [],
                floorList: [],
                isLoading: false,
                headerActive: false,
                isLogin: false,
              params: {
                categoryId: '',
                keyword: '',
                pageNum: 1,
                pageSize: 20,
                orderBy: 'DESC'
              }
            }
        },
        beforeCreate(){
            checkLogin().then((res)=>{
              res.status == 1 ? this.isLogin = false : this.isLogin = true
            })
        },
        created(){
            let followList = getStore('followList'),
                footprintList = getStore('footprintList')
            if(!followList){
                setStore('followList',[])
            }
            if(!footprintList){
                setStore('footprintList',[])
            }
          this.init()
        },
        mounted() {
            homeData().then((res) => {
              console.log(res)
                //this.headList = res.data.headList
              res.data.forEach((item)=>{
                item.img=require("../../assets/category/"+item.img)
              })
                this.categoryList = res.data

               // this.floorList = res.data.floorList
            })
            headData().then((res)=>{
              var imageHost="http://img.cdn.imbession.top/"
              res.data.forEach((item) => {
                this.headList.push({
                  imgUrl: imageHost + item.mainImage,
                  categoryId: item.categoryId
                })
              })
            })
            window.addEventListener('scroll', this.pageScroll)
        },
        methods: {
          ...mapMutations([
            'RECORE_FOOT'
          ]),
          productDetail: function (index) {
            let footprintList = getStore('footprintList')
            footprintList.unshift(this.productList[index])
            this.RECORE_FOOT(dedupeObject(footprintList))
            this.$router.push('./product/' + this.productList[index].id)
          },
          onInfinite(done) {
            this.params.pageNum++
            this.getProductList(this.params);
            done()
          },
          onRefresh(done) {
            this.params.pageNum = 1
            this.productList = []
            this.getProductList(this.params);
            done() // call done
          },
          init() {
            let keyword = " ",
              categoryId = ""
            this.productList = []
            this.params.categoryId = categoryId
            this.params.keyword = keyword
            this.keyword = keyword
            this.getProductList(this.params)
          },
          getProductList(params) {
            if (!this.keyword) {
              productListCategoryId(params).then((res) => {

                if (res.data.list.length === 0) {
                  this.$el.querySelector('.loading_text').style.display = 'none';
                  this.showMessage()
                  return
                }
                this.productList = this.productList.concat(res.data.list)
              })
            } else {
              productListKeyword(params).then((res) => {
                this.isLoading = false
                if (res.data.list.length === 0) {
                  this.$el.querySelector('.loading_text').style.display = 'none';
                  // alert('暂无更多数据！')
                  this.showMessage()
                  return
                }
                this.productList = this.productList.concat(res.data.list)
              })
            }
          },
            pageScroll() {
                let scrollTop = window.pageYOffset || document.documentElement.scrollTop || document.body.scrollTop
                scrollTop > 100 ? this.headerActive = true : this.headerActive = false
            },
            selectCategory(categoryId){
                this.$router.push('./product-list?categoryId='+categoryId)
            }
        },
        components: {
          refresh,
            mHeader,
            navBar,
            slider
        }
    }
</script>

<style lang="scss" scoped="" type="text/scss">
    @import '../../common/style/mixin';
    .home-header {
        background-color: black;
        position: fixed;
        left: 0;
        top: 0;
        @include fj;
        width: 100%;
        height: 100px;
        line-height: 100px;
        padding: 0 30px;
        @include boxSizing;
        font-size: 30px;
        color: #fff;
        z-index: 10000;
        &.active {
            /*<!--background: $red;-->*/
        }
        .icon-caidan {
            font-size: 50px;
        }
        .header-search {
            display: flex;
            width: 90%;
            height: 40px;
            line-height: 40px;
            margin: 20px 0;
            padding: 10px 0;
            color: #232326;
            background: #fff;
            @include borderRadius(40px);
            .app-name {
                padding: 0 20px;
                color: $red;
                font-size: 40px;
                font-weight: bold;
                border-right: 1px solid #666;
            }
            .icon-search {
                padding: 0 20px;
                font-size: 34px;
            }
            .search-title {
                font-size: 24px;
                color: #666;
            }
        }
        .icon-iconyonghu{
            font-size: 44px;
        }
    }

    .swiper-container .swiper_img {
        height: 400px;
    }

    .category-list {
        display: flex;
        flex-shrink: 0;
        flex-wrap: wrap;
        width: 100%;
        padding-bottom: 26px;
        div {
            display: flex;
            flex-direction: column;
            width: 20%;
            text-align: center;
            img {
                width: 80px;
                height: 80px;
                margin: 26px auto 16px auto;
            }
        }
    }
    .product-list {
      width: 100%;
      .product-item {
        @include fj;
        width: 100%;
        height: 240px;
        padding: 0px 0;
        border-bottom: 1px solid #dcdcdc;
        img {
          width: 280px;
          height: 240px;
          padding: 10px 20px;
          @include boxSizing;
        }
        .product-info {
          width: 56%;
          height: 240px;
          padding: 10px;
          @include boxSizing;
          .name {
            width: 100%;
            max-height: 80px;
            line-height: 40px;
            font-size: 30px;
            color: #333;
            overflow: hidden;
          }
          .subtitle {
            width: 100%;
            max-height: 40px;
            padding: 20px 0;
            line-height: 50px;
            font-size: 26px;
            color: #999;
            overflow: hidden;
          }
          .price {
            color: $red;
            font-size: 32px;
          }
        }
      }
    }
</style>
