<template>
  <div style="background-color: #F3f3f3;min-height: 100vh;" v-loading="loading" id="goodDetailsWrap">
    <div class="headerWrap" style="opacity: 0" id="headerWrap">
      <div class="detailsHeader">
        <div class="detailsIcon" @click="$router.back()">
          <van-icon name="arrow-left"/>
        </div>
        <div class="detailsHeaderContent" @click.stop="changeHeaderGood">
          <img src="../../assets/img/position.png" alt="" v-if="headerStatus">
          <p :class="{detailsHeaderContentActive:headerStatus == true}">商品</p>
        </div>
        <div class="detailsHeaderContent" @click.stop="changeHeaderDetails">
          <img src="../../assets/img/position.png" alt="" v-if="!headerStatus">
          <p :class="{detailsHeaderContentActive:headerStatus == false}">详情</p>
        </div>
      </div>
    </div>

    <div>
      <div class="detailsContent" v-if="goodDetails">
        <van-swipe @change="onChange" class="vanSwiper" v-if="goodDetails.goodsImages">
          <van-swipe-item v-for="(item,index) in imgList" @touchstart="Touchstart" @touchmove="Touchmove"
                          @touchend="Touchend(index)" :key="index">
            <img :src="item" alt="">
            <img class="goodListFalse" src="../../assets/img/goodEmpty.png" alt=""
                 v-if="goodDetails.goodsOptionStatus == '1' && changeDetails.subGoodsStoreEmpty == '0'">
            <img class="goodListFalse" src="../../assets/img/goodEmpty.png" alt=""
                 v-else-if="goodDetails.goodsOptionStatus == '0' && goodDetails.goodsStoreEmpty == '0'">
          </van-swipe-item>

          <div class="custom-indicator" slot="indicator">
            {{ current + 1 }}/{{imgList.length}}
          </div>
        </van-swipe>
        <div class="goodDitails">
          <div class="goodMount">
            <div class="goodPirceWrap">
              <div class="goodPirce" v-if="!goodDetails.goodsMarketProfit">
                <span>
                  <span>￥{{goodDetails.goodsMarketPrice}}</span>&nbsp;
                  <s v-if="goodDetails.goodsMarketPrice != goodDetails.goodsProductPrice">￥{{goodDetails.goodsProductPrice}}</s>
                </span>
                <span style="font-size: 0.3rem"
                      v-if="goodDetails.goodsPickStockNum">库存{{actionResult.id == '1' ? goodDetails.goodsStockNum : goodDetails.goodsPickStockNum}}件</span>
                <span style="font-size: 0.3rem" v-else>库存{{goodDetails.goodsStockNum}}件</span>
              </div>
              <div class="goodPirce" v-else>
                <span>￥{{goodDetails.goodsMarketPrice}}
                  <span v-if="goodDetails.goodsOptionStatus == '1'">&nbsp;~&nbsp;{{goodDetails.goodsMaxMarketPrice}}</span>
                  <span style=""><span
                    style="">&nbsp;/&nbsp;赚{{goodDetails.goodsMarketProfit}}</span><span
                    v-if="goodDetails.goodsOptionStatus == '1'">&nbsp;<span v-if="goodDetails.goodsMarketProfit != goodDetails.goodsMaxMarketProfit">~&nbsp;{{goodDetails.goodsMaxMarketProfit}}</span></span></span>
                </span>
                <span style="font-size: 0.3rem"
                      v-if="goodDetails.goodsPickStockNum">库存{{actionResult.id == '1' ? goodDetails.goodsStockNum : goodDetails.goodsPickStockNum}}件</span>
                <span style="font-size: 0.3rem" v-else>库存{{goodDetails.goodsStockNum}}件</span>
              </div>
            </div>
            <div class="goodDescription">
              <div class="goodDescriptionTitle">
                <div v-if="goodDetails.goodsLabel" style="display: inline-block">
                  <div class="goodLabels" v-if="goodDetails.goodsLabel.indexOf(',') != -1">
                    <span v-for="(item,index) in label">
                      <div class="lableInfo">{{item}}</div>
                    </span>
                  </div>
                  <div class="goodLabels" v-else>
                    <span>
                      <div class="lableInfo">{{goodDetails.goodsLabel}}</div>
                    </span>
                  </div>
                </div>
                {{goodDetails.goodsTitle}}&nbsp;·&nbsp;{{goodDetails.goodsSubtitle}}
              </div>
              <div class="goodCollect">
                <div v-if="goodDetails.goodsIsCollect == '1'" @click="cancelCollectGoodsOptionFalse">
                  <img src="../../assets/img/shop/shopping_collation_checkTrue.png" alt="">
                  <div class="collectText" style="color: #F2180C;">已收藏</div>
                </div>
                <div v-else @click="collectGoodsOptionFalse">
                  <img src="../../assets/img/shop/shopping_collation_checkFalse.png" alt="">
                  <div class="collectText">收藏</div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
      <div class="goodSpecifications" v-waves>
        <van-cell class="goodSpecificationsCell" is-link v-if="goodDetails.goodsPickStockNum" @click="showExpressWay = true">
          <div slot="title">
            配送方式：{{actionResult.name}}
          </div>
          <div slot='default'>{{actionResult.value}}</div>
        </van-cell>
        <van-cell title="配送方式" value="仅支持快递" class="goodSpecificationsCell" v-else/>
      </div>
      <div class="goodSpecifications" @click="showSku = true" v-waves>
        <van-cell is-link class="goodSpecificationsCell" id="teletextDetails">
          <template slot="title" class="goodSpecificationsCellContent">
            <span class="custom-text" style="color: #666;">已选：</span>
            <span class="custom-text goodSpecificationsCellContentText"
                  v-if="goodDetails.goodsOptionStatus == '1'">{{changeDetails.subGoodsSpecName}}<span
              v-if="changeDetails.subGoodsStoreEmpty != '0'">,{{buyNum}}件</span></span>
            <span class="custom-text goodSpecificationsCellContentText" v-else>{{buyNum}}&nbsp;件</span>
          </template>
        </van-cell>
      </div>

      <div class="teletextDetails" v-if="contentImgList && contentImgList.length">
        <div class="teletextDetailsHeader">
          <span>图文详情：</span>
        </div>
      </div>
      <div class="teletextDetailsContent" v-if="contentImgList && contentImgList.length">
        <img :src="item" alt="" v-for="(item,index) in contentImgList">
      </div>

      <div>
        <div v-if="goodDetails.goodsStoreEmpty != '0'">
          <div class="detailsFooter" v-if="goodDetails.goodsMarketProfit">
            <div class="footerButton" @click="$router.push({path:'/Main/Home'})">
              <img src="../../assets/img/main/home_iconCheckNromal.png" alt="">
            </div>
            <div class="footerButton" @click="addGoodsOptionFalseRouter">
              <img src="../../assets/img/main/shoppingCart_iconCheckNromal.png" alt="">
            </div>
            <div class="footerButton addshoppingTour" @click="showSku = true">
              <p>购买 <br> <span
                style="font-size: 0.3rem;display: inline-block;width: 100%;text-align: center">省{{goodDetails.goodsMarketProfit}}<span
                v-if="changeDetails.subGoodsMarketProfit">起</span></span></p>
            </div>
            <div class="footerButton buyTour" @click="animationWrapShow">
              <p>分享 <br> <span
                style="font-size: 0.3rem;display: inline-block;width: 100%;text-align: center">赚{{goodDetails.goodsMarketProfit}}<span
                v-if="changeDetails.subGoodsMarketProfit">起</span></span></p>
            </div>
          </div>
          <div class="detailsFooter" v-else>
            <div class="footerButton" @click="$router.push({path:'/Main/Home'})">
              <img src="../../assets/img/main/home_iconCheckNromal.png" alt="">
            </div>
            <div class="footerButton" @click="addGoodsOptionFalseRouter">
              <img src="../../assets/img/main/shoppingCart_iconCheckNromal.png" alt="">
            </div>
            <div class="footerButton addshopping" @click="showSku = true">
              <p>加入购物车</p>
            </div>
            <div class="footerButton buy" @click="showSku = true">
              <p>立即购买</p>
            </div>
          </div>
        </div>

        <div class="detailsFooter" v-else>
          <div class="footerButton addCollect" @click="collectGoodsOptionFalse"
               v-if="goodDetails.goodsIsCollect != '1'">
            <p>加入收藏</p>
          </div>
          <div class="footerButton addCollect" @click="cancelCollectGoodsOptionFalse" v-else>
            <p>取消收藏</p>
          </div>
        </div>
      </div>
    </div>

    <van-popup v-model="showSku" position="bottom" :overlay="true" lock-scroll>
      <div class="skuContnet">
        <div class="skuHeader">
          <div class="skuHeaderImgWrap">
            <img :src="changeDetails.subGoodsThumb" alt="" v-if="goodDetails.goodsOptionStatus == '1'">
            <img :src="imgList[0]" alt="" v-else>
          </div>
          <div v-if="goodDetails.goodsMarketProfit">
            <div class="skuHeaderDetails" v-if="goodDetails.goodsOptionStatus == '1'">
              <p>￥{{changeDetails.subGoodsMarketPrice}}&nbsp;/&nbsp;<span
                style="font-size: 0.34rem;">赚{{changeDetails.subGoodsMarketProfit}}</span></p>
              <div style="color: #999;">库存数量：{{changeDetails.subGoodsStockNum}}</div>
              <div>已选：<span>{{changeDetails.subGoodsSpecName}}</span></div>
            </div>
            <div class="skuHeaderDetails" v-else>
              <p>￥{{goodDetails.goodsMarketPrice}}&nbsp;/&nbsp;<span
                style="font-size: 0.34rem;">赚{{goodDetails.goodsMarketProfit}}</span></p>
              <div style="color: #999;">库存数量：{{goodDetails.goodsStockNum}}</div>
            </div>
          </div>
          <div v-else>
            <div class="skuHeaderDetails" v-if="goodDetails.goodsOptionStatus == '1'">
              <p>￥{{changeDetails.subGoodsMarketPrice}}&nbsp;<s style="color: #999;"
                                                                v-if="changeDetails.subGoodsMarketPrice != changeDetails.subGoodsProductPrice">￥{{changeDetails.subGoodsProductPrice}}</s>
              </p>
              <div style="color: #999;">库存数量：{{changeDetails.subGoodsStockNum}}</div>
              <div>已选：<span>{{changeDetails.subGoodsSpecName}}</span></div>
            </div>
            <div class="skuHeaderDetails" v-else>
              <p>￥{{goodDetails.goodsMarketPrice}}&nbsp;<s style="color: #999;"
                                                           v-if="goodDetails.goodsMarketPrice != goodDetails.goodsProductPrice">￥{{goodDetails.goodsProductPrice}}</s>
              </p>
              <div style="color: #999;">库存数量：{{goodDetails.goodsStockNum}}</div>
            </div>
          </div>
          <div class="skuHeaderClose" @click="showSku = false">
            <van-icon name="close"/>
          </div>
        </div>
        <div class="skuMiddle" v-if="goodDetails.goodsOptionStatus == '1'">
          <div v-for="(item,index) in goodDetails.goodSpecList">
            <p>{{item.specTitle}}</p>
            <div class="skuMiddleClass">
                <div :class="{skuMiddleClassSpanActive:detail.checkStatus == true}" class="skuMiddleClassSpan"
                     v-for="(detail,keyIndex) in item.specItemList"
                     @click="checkSubStatusClick(detail.itemId,index,keyIndex)">{{detail.itemTitle}}
                </div>
              </div>
          </div>

        </div>
        <div class="skuFooter">
          <van-cell-group>
            <van-cell :title="`购买数量`" v-if="goodDetails.goodsStoreEmpty != '0'" class="skuFooterCell">
              <div>
                <van-stepper
                  v-model="buyNum"
                  integer
                  :min="1"
                />
              </div>
            </van-cell>
          </van-cell-group>
          <div class="skuFooterButton" v-if="changeDetails.subGoodsStockNum != '0'">
            <div class="footerButton addshopping" style="width: 50%" @click="addGoodsOptionFalse">
              <p>加入购物车</p>
            </div>
            <div class="footerButton buy" style="width: 50%" @click="buyGoodsOptionFalse">
              <p>立即购买 <br>
                <span style="font-size: 0.3rem;display: inline-block;width: 100%;text-align: center"
                      v-if="changeDetails.subGoodsMarketProfit">省{{changeDetails.subGoodsMarketProfit}}</span>
                <span style="font-size: 0.3rem;display: inline-block;width: 100%;text-align: center"
                      v-else-if="goodDetails.goodsMarketProfit">省{{goodDetails.goodsMarketProfit}}</span>
              </p>
            </div>
          </div>
          <div class="skuFooterButton" v-else @click="collectGoodsOptionFalse">
            <div class="footerButton addshopping" style="width:100%;background-color: #Fdc734;color: #000;">
              <p>加入收藏</p>
            </div>
          </div>
        </div>
      </div>
    </van-popup>

    <van-action-sheet
      v-model="showExpressWay"
      :actions="actions"
      cancel-text="取消"
      @select="onSelect"
    />

    <div @click="animationWrapHidden" class="shareWrap" style="transform: scale(0)" id="shareWrap">
      <img src="../../assets/img/shareWrap.png" alt="">
    </div>

  </div>
</template>
<script>
  import { mapState, mapGetters } from 'vuex'
  import Vue from 'vue'
  import { ImagePreview } from 'vant'
  import * as common from '../../utils/common'
  import * as config from '../../utils/config'
  import * as wx from '../../utils/weixin'

  export default {
    data () {
      return {
        headerStatus: true,
        current: 0,
        showSku: false,
        buyNum: '1',
        imgList: [],
        contentImgList: [],
        checkSubStatus: '',
        changeDetails: [],
        loading: false,
        label: [],
        stop: false,
        showExpressWay: false,
        actions: [
          {
            name: '现场自提',
            id: '2',
            value:''
          },
          {
            name: '快递',
            id: '1',
            value:'运费6元，满100包邮'
          },
        ],
        actionResult: {
          name: '现场自提',
          id: '2',
          value:''
        },
        detailsOffset:'',
        headerWrapOffset:''
      }
    },
    computed: {
      ...mapState({
        'goodDetails': state => state.goodDetail.goodDetails,
      }),
      ...mapGetters(['getGoodSpecListChecked']),
    },
    watch: {
      getGoodSpecListChecked (v) {
        this.goodDetails.subGoodsList.filter(item => {
          if (item.subGoodsSpec == v) {
            this.changeDetails = item
          }
        })
      },
//      showSku(v){
//        if (v){
//          document.getElementById('goodDetailsWrap').style.overflowX = 'hidden'
//          document.getElementById('goodDetailsWrap').style.overflowY = 'hidden'
//        }else {
//          document.getElementById('goodDetailsWrap').style.overflowX = 'auto'
//          document.getElementById('goodDetailsWrap').style.overflowY = 'auto'
//        }
//      }
    },
    created () {
      this.loading = true
      this.$store.dispatch('getGoodDetails', this.$route.query.goodsId)
        .then(res => {
          this.loading = false
          let obj = {
            title: res.shareTitle,
            link: res.shareLink,
            imgUrl: res.shareIcon,
            desc: res.shareDescription,
          }
          setTimeout(() => {
            this.detailsOffset = document.getElementById('teletextDetails').offsetTop
            this.headerWrapOffset = document.getElementById('headerWrap').offsetTop
          },500)
          wx.initWx(obj)
          if (res.goodsImages) {
            this.imgList = this.goodDetails.goodsImages.split(',')
          }
          if (res.goodsContent) {
            this.contentImgList = this.goodDetails.goodsContent.split(',')
          }
          if (res.goodsLabel.indexOf(',') != -1) {
            this.label = this.goodDetails.goodsLabel.split(',')
          }
        })
        .catch(err => {
          this.loading = false
        })
    },
    mounted () {
      window.addEventListener('scroll', this.onLoad)

      common.getOpenId()
    },
    destroyed () {
      window.removeEventListener('scroll', this.onLoad)
      wx.shareInfo()
    },
    methods: {
      onLoad () {
        let scrollTop = document.documentElement.scrollTop || document.body.scrollTop
        scrollTop < this.detailsOffset ? this.headerStatus = true : this.headerStatus = false

        if (scrollTop > this.headerWrapOffset) {
          document.getElementById('headerWrap').style.opacity = '1'
        } else {
          document.getElementById('headerWrap').style.opacity = '0'
        }
      },
      changeHeaderGood () {
        if (document.getElementById('headerWrap').style.opacity == '0') {
          return
        }
        if (!this.headerStatus) {
          let timeOver = setInterval(() => {
            let osTop = document.documentElement.scrollTop || document.body.scrollTop
            let ispeed = Math.floor(-osTop / 5)
            document.documentElement.scrollTop = document.body.scrollTop = osTop + ispeed
            if (osTop === 0) {
              clearInterval(timeOver)
            }
          }, 30)
        }
      },
      changeHeaderDetails () {
        if (document.getElementById('headerWrap').style.opacity == '0') {
          return
        }
        if (this.headerStatus) {
          let timeOver = setInterval(() => {
            let osTop = document.documentElement.scrollTop || document.body.scrollTop
            let ispeed = Math.floor((this.detailsOffset) / 10)
            document.documentElement.scrollTop = document.body.scrollTop = osTop + ispeed
            if (osTop > (this.detailsOffset - 20)) {
              clearInterval(timeOver)
            }
          }, 30)
        }
      },
      onChange (index) {
        this.current = index
      },
      Touchstart (e) {
        this.stop = false
      },
      Touchmove (e) {
        this.stop = true
      },
      Touchend (index) {
        if (!this.stop) {
          ImagePreview({
            images: this.imgList,
            startPosition: index,
            showIndicators: true
          })
        }
      },
      onSelect (item) {
        this.actionResult = item
        this.showExpressWay = false
      },
      checkSubStatusClick (itemId, index, keyIndex) {
        let data = {
          itemId: itemId,
          index: index,
          keyIndex: keyIndex,
        }
        this.$store.dispatch('getCheckSubStatusClick', data)
      },
      buyGoodsOptionTrue () {
        this.$toast.loading({
          mask: true,
          message: '结算中...',
          loadingType: 'spinner',
          duration: 0
        })
        let goodsList = []
        goodsList.push({goodsId: this.changeDetails.subGoodsId, goodsNum: this.buyNum})
        let data = {
          packageId: '',
          goodsList: JSON.stringify(goodsList),
          storeStatus: window.location.pathname.length > 1 ? true : false
        }
        this.$store.dispatch('getPayCalculate', data)
          .then(res => {
            this.$router.push({
              path: '/ConfirmOrders',
              query: {
                packageId: data.packageId,
                goodsList: data.goodsList,
                status: '1',
                storeStatus: window.location.pathname.length > 1 ? true : false
              }
            })
            this.$toast.clear()
          })
          .catch(err => {
            this.$toast.clear()
          })
      },
      collectGoodsOptionFalse () {
        this.$toast.loading({
          mask: true,
          message: '加载中...',
          loadingType: 'spinner',
          duration: 0
        })
        this.$store.dispatch('getAddGood', this.goodDetails.goodsId)
          .then(res => {
            this.goodDetails.goodsIsCollect = '1'
          })
          .catch(err => {
            this.$toast.clear()
          })
      },
      cancelCollectGoodsOptionFalse () {
        this.$toast.loading({
          mask: true,
          message: '加载中...',
          loadingType: 'spinner',
          duration: 0
        })
        this.$store.dispatch('getCancelGood', this.goodDetails.goodsId)
          .then(res => {
            this.goodDetails.goodsIsCollect = '0'
          })
          .catch(err => {
            this.$toast.clear()
          })
      },
      addGoodsOptionFalse () {
        if (this.goodDetails.goodsPickStockNum && !this.actionResult.name) {
          common.toast('请选择配送方式')
          return
        }
        let data = {
          goodsId: this.changeDetails.subGoodsId || this.goodDetails.goodsId,
          goodsNum: this.buyNum,
          goodsEt: this.goodDetails.goodsPickStockNum ? this.actionResult.id : '1'
        }
        this.httpCli({
          url: config.URL_CART_PLUS,
          data: data
        })
          .then(res => {
            this.showSku = false
            if (res.errorCode == 100) {
              this.$toast.success('添加成功')
            } else {
//              this.$toast.fail('添加失败')
            }
          })
          .catch(err => {
            this.showSku = false
//            this.$toast.fail('添加失败')
          })
      },
      buyGoodsOptionFalse () {
        if (this.goodDetails.goodsPickStockNum && !this.actionResult.name) {
          common.toast('请选择配送方式')
          return
        }
        this.$toast.loading({
          mask: true,
          message: '结算中...',
          loadingType: 'spinner',
          duration: 0
        })
        let goodsList = []
        goodsList.push({
          goodsId: this.changeDetails.subGoodsId || this.goodDetails.goodsId,
          goodsNum: this.buyNum,
          goodsEt: this.goodDetails.goodsPickStockNum ? this.actionResult.id : '1'
        })
        let data = {
          goodsList: JSON.stringify(goodsList),
        }
        this.$store.dispatch('getPayCalculate', data)
          .then(res => {
            this.$router.push({path: '/ConfirmOrders'})
            this.$toast.clear()
          })
          .catch(err => {
            this.$toast.clear()
          })
      },
      addGoodsOptionFalseRouter () {
        this.$router.push({path: '/ShoppingCart'})
      },
      animationWrapShow () {
        document.getElementById('shareWrap').style.opacity = '1'
        document.getElementById('shareWrap').style.transform = 'scale(1)'
      },
      animationWrapHidden () {
        document.getElementById('shareWrap').style.opacity = '0'
        document.getElementById('shareWrap').style.transform = 'scale(0)'
      }
    }
  }
</script>
<style scoped>
  .headerWrap {
    position: fixed;
    width: 100%;
    top: 0px;
    z-index: 99;
    transition: all 0.5s;
  }

  .detailsHeader {
    height: 92px;
    /*border-bottom: 1PX #ddd solid;*/
    display: flex;
    justify-content: center;
    align-items: center;
    box-shadow: 0px 1PX 15px 3px #eee;
    background-color: #ffffff;
    position: relative;
  }

  .detailsIcon {
    position: absolute;
    left: 28px;
    height: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
    font-size: 32px;
    margin: 2px 0 0 2px;
  }

  .detailsHeaderContent {
    display: flex;
    justify-content: center;
    align-items: center;
    margin-right: 40px;
    margin-left: 40px;
    background-color: #ffffff;
  }

  .detailsHeaderContent img {
    width: 22px;
    margin-right: 10px;
  }

  .detailsHeaderContentActive {
    color: #F2180C;
  }

  .detailsContent {
    background-color: #ffffff;
  }

  .van-swipe {
    padding: 0px 0;
  }

  .van-swipe__track {
    position: relative;
  }

  .van-swipe-item {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 633px !important;
    position: relative;
  }

  .goodListFalse {
    position: absolute;
    left: 280px;
    top: 200px;
    width: 200px !important;
    height: 200px !important;
  }

  .vanSwiper img {
    height: 100%;
  }

  .custom-indicator {
    position: absolute;
    right: 40px;
    bottom: 40px;
    color: #999;
    height: 20px;
  }

  .goodDitails {
    display: flex;
    border-top: 1PX #ddd solid;
    padding: 0 25px 0 20px;
    box-sizing: border-box;
    justify-content: space-between;
  }

  .goodMount {
    width: 100%;
  }

  .vipCount {

  }

  .vipCount img {
    width: 29px;
  }

  .goodPirceWrap {
    margin: 49px 0 30px 0;
    font-size: 36px;
    color: #F2180C;
    display: flex;
    align-items: center;
  }

  .goodPirce {
    width: 100%;
    display: flex;
    align-items: center;
    justify-content: space-between;
  }

  .goodPirce span {
    display: inline-block;
    height:45px;
    line-height: 45px;
  }

  .goodPirceWrap s {
    font-size: 30px;
    color: #666;
    margin-right: 41px;
    margin-top: 3px;
  }

  .goodDescription {
    width: 100%;
    font-size: 30px;
    color: #333333;
    margin: 0px 0px 60px 8px;
    text-align: justify;
    display: flex;
    justify-content: space-between;
    align-items: flex-start;
  }

  .goodDescriptionTitle {

  }

  .goodDescriptionTitle .goodLabels {
    display: flex;
    justify-content: flex-start;
  }

  .goodDescriptionTitle .goodLabels span {
    border-radius: 20px;
    border: 1PX #F2180C solid;
    background-color: #F2180C;
    color: #fff !important;
    padding: 2px 10px;
    display: inline-block;
  }

  .lableInfo {
    font-size: 20px !important;
  }

  .goodCollect {
    font-size: 20px;
    color: #999;
    display: flex;
    flex-direction: column;
    align-content: center;
    white-space: nowrap;
    margin-left: 50px;
  }

  .goodCollect img {
    width: 40px;
    margin-bottom: 5px;
  }

  .goodCollect div {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
  }

  .collectText {
    width: 50px;
    text-align: center;
  }

  .goodSpecifications {
    margin-top: 10px;
  }

  .goodSpecificationsCell {
    padding: 40px 20px;
  }

  .goodSpecificationsCellContent {

  }

  .goodSpecificationsCell .van-cell__title {
    display: flex;
    align-items: center;
  }

  .goodSpecificationsCellContent span {
    font-size: 30px;
    color: #666;
  }

  .goodSpecificationsCellContentText {
    display: inline-block;
    color: #333333;
    width: 500px;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }

  .teletextDetails {
    margin-top: 10px;
    background-color: #ffffff;
  }

  .teletextDetailsHeader {
    padding: 40px 20px;
  }

  .teletextDetailsHeader span {
    font-size: 30px;
    color: #666;
  }

  .teletextDetailsContent {
    padding-bottom: 110px;
    font-size: 0px;
  }

  .teletextDetailsContent img {
    width: 100%;
  }

  .detailsFooter {
    position: fixed;
    left: 0;
    bottom: 0;
    width: 100%;
    height: 110px;
    display: flex;
    border-top: 1PX #ddd solid;
    background-color: #ffffff;
  }

  .footerButton {
    width: 15%;
    display: flex;
    justify-content: center;
    align-items: center;
    position: relative;
  }

  .footerButton::before {
    content: " ";
    position: absolute;
    top: 50%;
    left: 50%;
    opacity: 0;
    width: 100%;
    height: 100%;
    border: inherit;
    border-color: #000;
    background-color: #000;
    border-radius: inherit;
    -webkit-transform: translate(-50%, -50%);
    transform: translate(-50%, -50%);
  }

  .footerButton:active::before {
    opacity: 0.3;
  }

  .footerButton img {
    width: 50px;
  }

  .addshopping {
    width: 35%;
    font-size: 30px;
    background-color: #F2180C;
    color: #ffffff;
  }

  .addshoppingTour {
    height: 80%;
    text-align: center;
    border-radius: 50px;
    width: 35%;
    font-size: 30px;
    background-color: rgb(168, 49, 59);
    color: #ffffff;
    margin: 10px 20px 0 0;
  }

  .buy {
    width: 35%;
    font-size: 30px;
    background-color: #333;
    color: #ffffff;
  }

  .buyTour {
    height: 80%;
    border-radius: 50px;
    width: 35%;
    text-align: center;
    font-size: 30px;
    background-color: #F2180C;
    color: #ffffff;
    margin-top: 10px;
  }

  .addCollect {
    font-size: 30px;
    width: 100%;
    color: #000;
    background-color: #Fdc734;
  }

  .van-popup {
    overflow-y: visible;
  }

  .skuContnet {
    background-color: #ffffff;
    padding: 0 20px;
    width: 100%;
    box-sizing: border-box;
    transition: all 0.3s !important;
  }

  .skuHeader {
    height: 180px;
    width: 100%;
    position: relative;
  }

  .skuHeaderImgWrap {
    float: left;
    position: relative;
    width: 230px;
    height: 230px;
    display: flex;
    justify-content: center;
    align-items: center;
    background-color: #ffffff;
    border-radius: 15px;
    margin-top: -50px;
  }

  .skuHeaderImgWrap img {
    width: 100%;
    height: 230px;
    border-radius: 15px;
  }

  .skuHeaderDetails {
    position: absolute;
    left: 280px;
    top: 10px;
  }

  .skuHeaderDetails p {
    height: 20px;
    font-size: 34px;
    color: #F2180C;
    display: flex;
    align-items: center;
  }

  .skuHeaderDetails div {
    display: flex;
    align-items: center;
  }

  .skuHeaderDetails div span:nth-child(2) {
    margin-top: 4px;
  }

  .skuHeaderClose {
    position: absolute;
    right: 10px;
    top: 30px;
    font-size: 30px;
  }

  .skuMiddle {
    padding: 30px 0 10px 0;
  }

  .skuMiddle p {
    color: #666;
    font-size: 30px;
  }

  .skuMiddleClass {
    min-width: 100%;
    overflow: hidden;
  }

  .skuMiddleClassSpan {
    white-space: nowrap;
    padding: 0 25px;
    margin:0px 20px 20px 0;
    /*width: 170px;*/
    height: 64px;
    font-size: 30px;
    border-radius: 32px;
    border: 2px #999999 solid;
    color: #999;
    float: left;
    display: flex;
    justify-content: center;
    align-items: center;
    position: relative;
  }

  .skuMiddleClassSpan:nth-child(1) {
    margin-left: 0px !important;
  }

  .skuMiddleClassSpan::before {
    content: " ";
    position: absolute;
    top: 50%;
    left: 50%;
    opacity: 0;
    width: 100%;
    height: 100%;
    border: inherit;
    border-color: #000;
    background-color: #000;
    border-radius: inherit;
    -webkit-transform: translate(-50%, -50%);
    transform: translate(-50%, -50%);
  }

  .skuMiddleClassSpan:active::before {
    opacity: 0.3;
  }

  .skuMiddleClassSpanActive {
    border: 2px #F2180C solid;
    color: #F2180C;
  }

  .skuMiddleClassSpanFail {
    color: #cbcbcb;
    background-color: #f3f3f3;
    border: 0px;
    /*cursor: not-allowed;*/
  }

  .skuMiddleClassSpanFail:active::before {
    opacity: 0;
  }

  .skuFooter {
    padding:15px 0 25px 0;
    border-top: 1PX #ddd solid;
    margin-left: -20px;
    margin-top: 10px;
    width: calc(100% + 40px);
  }

  .skuFooterCell::after{
    border: 0px !important;
  }
  .van-hairline--top-bottom::after{
    border: 0px !important;
  }

  .skuFooterButton {
    display: flex;
    height: 100px;
    background-color: #F2180C;
    color: #ffffff;
    margin-bottom: -26px;
    margin-top: 10px;
  }
</style>
