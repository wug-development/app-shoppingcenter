<template>
    <div id="MyOrderDetail" class="myorderdetail">
        <Header headtitle="订单详情"></Header>
        <div class="addr-box">
            <div class="namephone">
                <span>收货人：{{orderinfo.shouhuoren}}</span>
                <span>{{orderinfo.shouhuomobile}}</span>
            </div>
            <div class="addr-info">
                收货地址：{{orderinfo.shouhuoaddress}}
            </div>
        </div>

        <div class="products-box">
            <div class="storename">Lovzvzu 商城</div>
            <ul class="product-list">
                <li v-for="(p, index) in orderinfo.orderProductList" :key="index">
                    <div class="proimg">
                        <img :src="p.productcoverimage" alt="">
                    </div>
                    <div class="proinfo">
                        <div class="name">{{p.productname}}</div>
                        <div class="attr">{{p.color}} {{p.size}}</div>
                        <div class="pnum"><span class="price">&yen;{{p.productprice}}</span><span>&#120;{{p.buynumber}}</span></div>
                    </div>
                </li>
            </ul>
            <div class="delivery">
                <span>配送方式</span>
                <span>快递免运费</span>
            </div>
        </div>

        <div class="totalprice">
            <ul class="price-item">
                <li><span>商品金额</span><span>&yen;{{orderinfo.price}}</span></li>
                <li v-if="orderinfo.usecredit == 1"><span>积分</span><span>-&yen;{{credit.madeCredit.usercreditnumber}}.00</span></li>
                <li><span>运费</span><span>+&yen;0.00</span></li>
            </ul>
            <div class="total">
                合计：<span>&yen;{{orderinfo.price}}</span>
            </div>
        </div>
        <div class="totalbox" v-if="orderinfo.status == 1">
            <div class="amount">合计：<span>&yen;{{orderinfo.price}}</span></div>
            <div class="btn" @click="wxPay">去支付</div>
        </div>
    </div>
</template>

<script>
import Header from '@/components/common/Header.vue'

export default {
    name: 'MyOrderDetail',
    data () {
        return {
            orderinfo: {}
        }
    },
    components: {
        Header
    },
    beforeMount () {
        this.$store.state.menu = false

        let info = sessionStorage.getItem('orderitem')
        this.orderinfo = JSON.parse(info)
        console.log(this.orderinfo)
    },
    methods: {
        wxPay: function () {
            this.$http.get(this.apis + '/payWeixin/wxPayByWeb', {params: {
                out_trade_no: this.orderinfo.orderno
            }})
            .then(res => {
                if (res && res.data && res.data.code === 1) {
                    this.Toast('微信支付调用成功')
                    this.Indicator.close()
                    this.isSubmited = false
                    jsSDK(res.data.obj, this)
                } else {
                    this.Toast('微信支付调用失败')
                    this.Indicator.close()
                    this.isSubmited = false
                }
            })
            .catch(e => {
                this.Indicator.close()
                this.Toast('微信支付调用失败')
                this.isSubmited = false
            })
        }
    }
}

function jsSDK (params, vue) {
    if (typeof window.WeixinJSBridge === 'undefined') {
        if (document.addEventListener) {
            document.addEventListener('WeixinJSBridgeReady', function () { onBridgeReady(params, vue) }, false)
        } else if (document.attachEvent) {
            document.attachEvent('WeixinJSBridgeReady', function () { onBridgeReady(params, vue) })
            document.attachEvent('onWeixinJSBridgeReady', function () { onBridgeReady(params, vue) })
        }
    } else {
        onBridgeReady(params, vue)
    }
}

function onBridgeReady (res, vue) {
    window.WeixinJSBridge.invoke(
        'getBrandWCPayRequest', {
            'appId': res.appId, // 公众号名称，由商户传入
            'timeStamp': res.timeStamp, // 时间戳，自1970年以来的秒数
            'nonceStr': res.nonceStr, // 随机串
            'package': res.packageValue,
            'signType': res.signType, // 微信签名方式：
            'paySign': res.paySign // 微信签名
        },
        function (res) {
            console.log(res)
            if (res.err_msg === 'get_brand_wcpay_request:ok') {
                vue.Toast('支付成功')
                vue.$router.push({
                    path: '/my'
                })
            } else if (res.err_msg === 'get_brand_wcpay_request:cancel') {
                vue.Toast('取消支付')
            } else {
                vue.Toast('支付失败')
            }
        }
    )
}
</script>

<style lang="scss">
    .myorderdetail{
        height: 100%;
        padding-top: .9rem;
        background-color: #f9f9f9;
        .header-box{
            box-shadow: none;
        }
        .addr-box{
            padding: .2rem .6rem;
            background-position-x: 7rem;
            background-size: .32rem;
            background-color: #fff;
            border-bottom: .04rem dashed #777;
            .namephone{
                display: flex;
                justify-content: space-between;
                color: #000;
                font-size: .28rem;
                font-family: '黑体', sans-serif;
                font-weight: bold;
                position: relative;
            }
            .namephone::before{
                content: '';
                position: absolute;
                top: 0;
                left: -.4rem;
                width: .4rem;
                height: .4rem;
                background: url('../../assets/images/site.png') no-repeat center;
                background-size: .24rem;
            }
            .addr-info{
                margin-top: .1rem;
                color: #777;
            }
        }
        .products-box{
            margin-top: .2rem;
            .storename{
                height: .8rem;
                line-height: .8rem;
                color: #333;
                padding-left: .6rem;
                background: url('../../assets/images/store.png') no-repeat center;
                background-position-x: .3rem;
                background-size: .26rem;
                background-color: #fff;
            }
            .product-list{
                padding: 0 .2rem;
                box-sizing: border-box;
                li{
                    display: flex;
                    padding: .2rem 0;
                    box-sizing: border-box;
                    .proimg {
                        width: 1.4rem;
                        height: 1.4rem;
                        img{
                            display: block;
                            width: 100%;
                            height: 100%;
                        }
                    }
                    .proinfo{
                        width: 5.7rem;
                        padding-left: .2rem;
                        box-sizing: border-box;
                        .name{
                            font-weight: bold;
                        }
                        .attr{
                            color: #777;
                        }
                        .pnum{
                            margin-top: .1rem;
                            display: flex;
                            justify-content: space-between;
                            .price{
                                color: #f00;
                            }
                        }
                    }
                }
            }
            .delivery{
                background-color: #fff;
                padding: 0 .2rem;
                display: flex;
                height: .8rem;
                line-height: .8rem;
                display: flex;
                justify-content: space-between;
            }
        }
        .totalprice{
            margin-top: .2rem;
            background-color: #fff;
            padding-left: .2rem;
            box-sizing: border-box;
            .price-item{
                padding: .1rem .2rem .1rem 0;
                border-bottom: .02rem solid #ddd;
                li{
                    display: flex;
                    justify-content: space-between;
                    font-size: .2rem;
                    line-height: .5rem;
                }
            }
            .total{
                text-align: right;
                color: #777;
                height: .8rem;
                line-height: .8rem;
                padding-right: .2rem;
                box-sizing: border-box;
                span{
                    color: #f00;
                }
            }
        }
        .totalbox{
            position: absolute;
            left: 0;
            bottom: 0;
            width: 100%;
            height: .9rem;
            line-height: .9rem;
            background-color: #fff;
            display: flex;
            font-size: .28rem;
            .amount{
                width: 5.5rem;
                text-align: right;
                color: #333;
                border-top: .02rem solid #eee;
                box-sizing: border-box;
                padding-right: .2rem;
                span{
                    color: #f00;
                }
            }
            .btn{
                width: 2rem;
                text-align: center;
                color: #fff;
                background-color: #f00;
            }
            .btn:active{
                background-color: #DA0000;
            }
        }
    }
</style>
