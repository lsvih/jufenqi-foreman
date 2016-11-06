<template>
<div>
    <header>
        <tab active-color='#88C929' :index.sync="index">
            <tab-item active-class="tab-active" :selected="index === 0" v-tap="index = 0">装修订单</tab-item>
            <tab-item active-class="tab-active" :selected="index === 1" v-tap="index = 1">已完成</tab-item>
        </tab>
    </header>

    <swiper :index.sync="index" :height="getScreenHeight()+'px'" :show-dots="false">
        <swiper-item height="100%">
            <div class="tab-swiper vux-center content">
                <scroller :height="getScreenHeight()-44+'px'" lock-x scroller-y v-ref:zx>
                    <div>
                        <div class="order" v-for="order in zxList" v-tap="viewDetail('zx',order.orderNo,order.plan.id)">
                            <img :src="order.customerImage">
                            <div class="tel" onclick="location.href='tel:{{order.customerMobile}}'">{{order.customerMobile}}</div>
                            <div class="name">{{order.customerName}}</div>
                            <div class="status">{{zxStatusList[order.plan.status].name}}</div>
                        </div>
                    </div>
                    <div v-if="zxList.length==0">
                        <div class="no-data"><img src="no-data.png"><span>暂无订单</span></div>
                    </div>
                </scroller>
            </div>
        </swiper-item>
        <swiper-item height="100%">
            <div class="tab-swiper vux-center content">
                <scroller :height="getScreenHeight()-44+'px'" lock-x scroller-y v-ref:wc>
                    <div>
                        <div class="order" v-for="order in wcList" v-tap="viewDetail('zx',order.orderNo)">
                            <img :src="order.customerImage">
                            <div class="tel" onclick="location.href='tel:{{order.customerMobile}}'">{{order.customerMobile}}</div>
                            <div class="name">{{order.customerName}}</div>
                            <div class="status">{{zxStatusList[order.plan.status].name}}</div>
                        </div>
                    </div>
                    <div v-if="wcList.length==0">
                        <div class="no-data"><img src="no-data.png"><span>暂无订单</span></div>
                    </div>
                </scroller>
            </div>
        </swiper-item>
    </swiper>
</div>
</template>

<script>
import Lib from 'assets/Lib.js'
import {
  Tab,
  TabItem
} from 'vux-components/tab'
import Swiper from 'vux-components/swiper'
import SwiperItem from 'vux-components/swiper-item'
import Scroller from 'vux-components/scroller'
import axios from 'axios'
try{
  axios.defaults.headers.common['x-user-token'] = JSON.parse(localStorage.getItem("user")).token
}catch(e){
  localStorage.clear()
  window.location.href = `./wxAuth.html?url=index.html`
}
export default {
    data() {
        return {
            index: 0,
            zxStatusList: [{
                status: 0,
                name: "订单已删除"
            }, {
                status: 1,
                name: "已预约"
            }, {
                status: 2,
                name: "已上门"
            }, {
                status: 3,
                name: "待选方案"
            }, {
                status: 4,
                name: "待支付"
            }, {
                status: 5,
                name: "待施工"
            }, {
                status: 6,
                name: "施工中"
            }, {
                status: 7,
                name: "已完工"
            }, {
                status: 8,
                name: "订单已取消"
            }],
            zxList: [],
            wcList: [],
        }
    },
    components: {
      Tab,
      TabItem,
      Swiper,
      SwiperItem,
      Scroller
    },
    ready() {
        let suc_count = 0
        this.index = (Lib.M.GetRequest().type - 1) || 0
        axios.get(`${Lib.C.orderApi}decorationPlans`, {
            params: {
                filter: `foremanId:${JSON.parse(window.localStorage.getItem('user')).userId}|status:[1,7]`
            }
        }).then((res) => {
            res.data.data.map((e) => {
                if (e.status == 7) {
                    this.wcList.push(e)
                } else {
                    this.zxList.push(e)
                }
            })
            this.$refs.zx.reset()
            this.$refs.wc.reset()
        }).catch((res) => {
            console.log(res)
            alert("获取订单失败，请稍候再试QAQ")
        })
    },
    methods: {
      getScreenHeight() {
        return document.body.clientHeight
      },
        getTime(timeStamp) {
            var d = new Date(timeStamp * 1000);
            var Y = d.getFullYear() + '-';
            var M = (d.getMonth() + 1 < 10 ? '0' + (d.getMonth() + 1) : d.getMonth() + 1) + '-';
            var D = (d.getDate() < 10 ? '0' + (d.getDate()) : d.getDate());
            return Y + M + D
        },
        viewDetail(type, orderNo,planId) {
            eval(`window.location.href='${type}-order.html?orderNo=${orderNo}&planId=${planId}'`)
        }
    }
}
</script>

<style>
html {
    height: 100%;
}

body {
    background-color: #eee;
    height: 100%;
}
</style>
<style scoped lang="less">
.content {
    padding-top: 45px;
}
header {
    position: fixed;
    height: 44px;
    width: 100%;
    left: 0;
    top: 0;
    z-index: 30;
}
.tab-active {
    color: #88C929 !important;
    border-color: #88C929 !important;
}
.order {
    width: calc(~"100% - 15px");
    height: 80px;
    background-color: #fff;
    padding-left: 15px;
    position: relative;
    border-bottom: 1px solid #eee;
    img {
        position: absolute;
        top: 10px;
        left: 15px;
        width: 60px;
        height: 60px;
    }
    .status {
        position: absolute;
        font-size: 12px;
        color: #393939;
        bottom: 10px;
        left: 85px;
    }
    .name {
        position: absolute;
        font-size: 12px;
        color: #393939;
        top: 10px;
        left: 86px;
    }
    .tel {
        position: absolute;
        right: 15px;
        top: 0;
        height: 80px;
        line-height: 80px;
        color: #3BA794;
    }
}
.no-data {
    position: relative;
    width: 100%;
    img {
        position: absolute;
        top: 220px;
        left: calc( ~"50% - 35.5px" );
        height: 71px;
        width: 71px;
    }
    span {
        display: block;
        position: absolute;
        width: 100%;
        height: 16px;
        font-size: 16px;
        line-height: 16px;
        left: 0;
        top: 307px;
        color: #DADADA;
        text-align: center;
    }
}
</style>
