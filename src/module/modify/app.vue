<template>
<group title="请完成您的设计方案:">
    <x-input :value.sync="order.plan.price" title="请输入方案价格"></x-input>
</group>
<group title="请输入您的方案描述">
    <x-textarea :value.sync="order.plan.description" placeholder="方案描述"></x-textarea>
</group>
<group title="您的方案已有的图片" v-if="order.plan.images.length!=0">
    <div class="plan-image" v-for="image in order.plan.images">
        <img class="image" :src="imgUrl + image">
        <img src="./del.png" class="del" v-tap="delExist(image,$index)">
    </div>
</group>
<group title="您即将上传的方案图片" v-if="addImages.length!=0">
    <div class="plan-image" v-for="image in addImages">
        <img class="image" :src="image.local">
        <img src="./del.png" class="del" v-tap="delAdd($index)">
    </div>
</group>
<group>
    <j-to-upload-photo title="请上传您的方案图片" value="点击上传或拍摄您的|方案图片" v-tap="getInfo()"></j-to-upload-photo>
</group>
<div class="status-3-btn" :class="{'active':isFilled()}" v-tap="isFilled()?submit():return">
    <div class="btn-right">确认修改</div>
</div>
</template>

<script>
import Lib from 'assets/Lib.js'
import Group from 'vux-components/group'
import Cell from 'vux-components/cell'
import XInput from 'vux-components/x-input'
import XTextarea from 'vux-components/x-textarea'
import JToUploadPhoto from 'components/JToUploadPhoto.vue'
import axios from 'axios'
try {
  let now = Number(new Date().getTime())
  if (Number(JSON.parse(localStorage.user).expiredAt) < now) {
    localStorage.removeItem('user')
    location.href = './wxAuth.html?url=' + encodeURIComponent(location.href)
  }
  axios.defaults.headers.common['Authorization'] = JSON.parse(localStorage.getItem("user")).tokenType + ' ' + JSON.parse(localStorage.getItem("user")).token
} catch (e) {
  localStorage.clear()
  window.location.href = `./wxAuth.html?url=index.html`
}
export default {
    data() {
        return {
            addImages: [],
            delImages: [],
            order: {},
            imgUrl: Lib.C.imgUrl
        }
    },
    components: {
        Group,
        Cell,
        XInput,
        XTextarea,
        JToUploadPhoto
    },
    ready() {
        axios.post(`${Lib.C.wxApi}qy/jsapiTicket`, {url:location.href}, {
            withCredentials: true,
            responseType: 'json'
        }).then((res) => {
            wxReady(res.data.data)
        }).catch((res) => {
            alert("网络连接失败，请刷新重试")
            console.log(res.data.data)
        })
        axios.get(`${Lib.C.orderApi}decorationPlans/${Lib.M.GetRequest().planId}`).then((res) => {
            this.order = res.data.data
        }, (res) => {
            alert("获取订单失败，请稍候再试QAQ")
        })
    },
    methods: {
        getInfo() {
            let that = this
            wx.chooseImage({
                count: 1, // 默认9
                sizeType: ['original', 'compressed'],
                sourceType: ['camera', 'album'], // 可以指定来源是相册还是相机，默认二者都有
                success: function(res) {
                    let thisId = res.localIds[0]
                    wx.uploadImage({
                        localId: thisId, // 需要上传的图片的本地ID，由chooseImage接口获得
                        isShowProgressTips: 1, // 默认为1，显示进度提示
                        success: function(res) {
                            var serverId = res.serverId // 返回图片的服务器端ID
                            that.addImages.push({
                                local: thisId,
                                server: serverId
                            })
                        }
                    })
                }
            })
        },
        delExist(imageName, index) {
            this.delImages.push(imageName)
            this.order.plan.images.splice(index, 1)
        },
        delAdd(index) {
            this.addImages.splice(index, 1)
        },
        submit() {
            let upImg = []
            this.addImages.map((e) => {
                upImg.push(e.server)
            })
            axios.put(`${Lib.C.orderApi}decorationPlans/${Lib.M.GetRequest().planId}`, {
                "description": this.order.plan.description,
                "price": Number(this.order.plan.price),
                "deletedImages": this.delImages,
                "newImages": [],
                "newWechatImages": upImg
            }, {
                withCredentials: true,
                responseType: 'json'
            }).then((res) => {
                alert("更新设计方案成功")
                location.href = './index.html'
            }).catch((res) => {
                alert("网络连接失败，请重试")
            })
        },
        isFilled(){
          return !!(this.addImages.length>0&&this.order.plan.price)
        }
    }
}

function wxReady(obj) {
    obj.debug = false
    obj.jsApiList = ["chooseImage", "previewImage", "uploadImage", "downloadImage"]
    wx.config(obj)
}
</script>

<style>
body {
    background-color: #eee;
}
</style>
<style lang="less">
.plan-image {
    position: relative;
    width: calc(~"100% - 30px");
    margin-left: 15px;
    height: auto;
    margin-bottom: 10px;
    .image {
        width: 100%;
        height: auto;
    }
    .del {
        height: 27px;
        width: 27px;
        position: absolute;
        right: 10px;
        top: 10px;
        z-index: 2;
    }
}
.status-3-btn {
    position: relative;
    width: calc(~"100% - 30px");
    margin-left: 15px;
    border-radius: 2px;
    height: 44px;
    margin-top: 10px;
    background-color: rgb(226, 226, 226);
    color: #fff;
    line-height: 44px;
    text-align: center;
}
.active{
  background-color: rgb(158, 188, 43);
}
</style>
