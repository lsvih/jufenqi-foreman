<template>

</template>

<script>
import Lib from 'assets/Lib.js'
import axios from 'axios'
export default {
    data() {
        return {
            lastUrl: Lib.M.GetRequest().url ? unescape(Lib.M.GetRequest().url) : './index.html',
            code: Lib.M.GetRequest().code,
            appId: "wx532853a12ecebf6b"
        }
    },
    ready() {
        if (Lib.M.isAuth()) {
            location.href = this.lastUrl
        } else {
            this.getCode()
        }
    },
    methods: {
        getCode() {
            if (this.code) {
                axios.post(`${Lib.C.userApi}auth/loginUsingWechatQy`, {}, {
                    params: {
                        code: this.code
                    },
                    withCredentials: true,
                    responseType: 'json'
                }).then((res) => {
                    window.localStorage.setItem("user", JSON.stringify(res.data.data))
                    location.href = this.lastUrl
                }).catch((res) => {
                    alert("微信登录失败，请稍后重试")
                    console.log(res)
                    return false;
                })
            } else {
                let a = `https://open.weixin.qq.com/connect/oauth2/authorize?appid=${this.appId}&redirect_uri=${location.href}&response_type=code&scope=SCOPE&state=STATE#wechat_redirect`
                location.href = a
            }
        }
    }
}
</script>
