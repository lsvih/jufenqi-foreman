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
                    <no-data v-if="zxList.length==0"></no-data>
                    <div v-else>
                        <j-order-block v-for="order in zxList" v-tap="viewDetail('zx',order.orderNo,order.plan.id)" :img="order.customerImage" :name="order.customerName" :tel="order.customerMobile" :status="Status.zx[order.plan.status].name"></j-order-block>
                    </div>
                </scroller>
            </div>
        </swiper-item>
        <swiper-item height="100%">
            <div class="tab-swiper vux-center content">
                <scroller :height="getScreenHeight()-44+'px'" lock-x scroller-y v-ref:wc>
                    <no-data v-if="wcList.length==0"></no-data>
                    <div v-else>
                        <j-order-block v-for="order in wcList" v-tap="viewDetail('zx',order.orderNo,order.plan.id)" :img="order.customerImage" :name="order.customerName" :tel="order.customerMobile" :status="Status.zx[order.plan.status].name"></j-order-block>
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
import JOrderBlock from 'common/components/j-order-block'
import NoData from 'common/components/no-data'
import axios from 'axios'
import Status from 'common/status'
try {
    axios.defaults.headers.common['x-user-token'] = JSON.parse(localStorage.getItem("user")).token
} catch (e) {
    localStorage.clear()
    window.location.href = `./wxAuth.html?url=index.html`
}
export default {
    data() {
        return {
            index: 0,
            zxList: [],
            wcList: [],
            Status
        }
    },
    components: {
        Tab,
        TabItem,
        Swiper,
        SwiperItem,
        Scroller,
        JOrderBlock,
        NoData
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
        viewDetail(type, orderNo, planId) {
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
</style>
