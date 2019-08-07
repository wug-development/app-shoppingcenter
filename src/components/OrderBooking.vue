<template>
    <div id="OrderBooking" class="orderbooking-box">
        <div class="addr-box">
            <template v-if="isAddr">
                <div @click="toAddrList(addrInfo.id)">
                    <div class="namephone">
                        <span>收货人：{{addrInfo.name}}</span>
                        <span>{{addrInfo.mobile}}</span>
                    </div>
                    <div class="addr-info">
                        收货地址：{{addrInfo.province}} {{addrInfo.city}} {{addrInfo.district}} {{addrInfo.address}}
                    </div>
                </div>
            </template>
            <template v-if="!isAddr">
                <div class="newaddr" @click="toNewAddr">
                    <span class="btn"></span><span>选择收货地址</span>
                </div>
            </template>
        </div>

        <div class="products-box">
            <div class="storename">Lovzvzu 商城</div>
            <ul class="product-list">
                <li v-for="(item, index) in dataList" :key="index">
                    <div class="proimg">
                        <img :src="item.pic" alt="">
                    </div>
                    <div class="proinfo">
                        <div class="name">{{item.name}}</div>
                        <div class="attr">{{item.selModel.color.productcolorValue}} {{item.selModel.color.productsizeValue}}</div>
                        <div class="pnum"><span class="price">&yen;{{item.price}}</span><span>&#120;{{item.num}}</span></div>
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
                <li><span>商品金额</span><span>&yen;{{totalPrice}}</span></li>
                <li><span>运费</span><span>+&yen;0.00</span></li>
            </ul>
            <div class="total">
                合计：<span>{{totalPrice}}</span>
            </div>
        </div>

        <div class="totalbox">
            <div class="amount">合计：<span>&yen;{{totalPrice}}</span></div>
            <div class="btn" @click="submitOrder">提交订单</div>
        </div>
    </div>
</template>

<script>
export default {
    name: 'OrderBooking',
    data () {
        return {
            isAddr: 0,
            buytype: '',
            dataList: [],
            totalPrice: 0,
            addrInfo: {},
            isSubmited: false
        }
    },
    methods: {
        toNewAddr: function () {
            this.$router.push({
                path: '/my/newaddress'
            })
        },
        toAddrList: function (id) {
            this.$router.push({
                path: '/my/myaddress?id=' + id + '&ty=' + this.buytype
            })
        },
        submitOrder: function () {
            if (this.isSubmited) {
                return false
            } else if (this.addrInfo && this.addrInfo.id) {
                this.Indicator.open()
                this.isSubmited = true
                let pids = ''
                let nums = ''
                for (let i in this.dataList) {
                    if (i > 0) {
                        pids = pids + ','
                        nums = nums + ','
                    }
                    pids += this.dataList[i].id
                    nums += this.dataList[i].num
                }

                this.$http.get(this.apis + '/order/addOrderAjaxNew', {params: {
                    openid: sessionStorage.getItem('openID'),
                    address: this.addrInfo.id,
                    productIds: pids,
                    buyproductNumbers: nums,
                    productcolorsizestocks: [],
                    useCredit: 0,
                    useCreditNumber: 0
                }})
                .then(res => {
                    console.log(res)
                    if (res && res.data) {
                        if (res.data.code === 1) {
                            console.log(res.data.obj)
                            this.wxPay(res.data.obj[0])
                        } else if (res.data.code === 2) {
                            this.$messagebox('库存不足')
                            this.Indicator.close()
                            this.isSubmited = false
                        } else if (res.data.code === 3) {
                            this.$messagebox('下单成功')
                            this.Indicator.close()
                            this.isSubmited = false
                            this.$router.push({
                                path: '/my'
                            })
                        } else {
                            this.$messagebox(res.data.msg)
                            this.Indicator.close()
                            this.isSubmited = false
                        }
                    } else {
                        this.$messagebox('提交失败')
                        this.Indicator.close()
                        this.isSubmited = false
                    }
                })
                .catch(e => {
                    this.Indicator.close()
                    this.isSubmited = false
                })
            } else {
                this.$messagebox('请选择收货地址')
                this.isSubmited = false
            }
        },
        getAddr: function () {
            this.$http.get(this.apis + '/address/getCustomerAddressByOpenIdAjax', {params: {
                openId: sessionStorage.getItem('openID')
            }})
            .then(res => {
                if (res && res.data && res.data.code === 1) {
                    let _data = res.data.obj
                    for (let i in _data) {
                        if (_data[i].status === 1) {
                            this.addrInfo = _data[i]
                            this.isAddr = true
                        }
                    }
                }
            })
        },
        wxPay: function (obj) {
            this.$http.get(this.apis + '/payWeixin/wxPayByWeb', {params: {
                out_trade_no: obj.orderno
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
    },
    beforeMount () {
        this.$store.state.menu = false
        this.buytype = this.$route.query.ty
        if (this.buytype === 'cart') {
            let pros = sessionStorage.getItem('cart')
            if (pros) {
                this.dataList = JSON.parse(pros)
                for (let i in this.dataList) {
                    this.totalPrice = keepTwoDecimal(this.totalPrice + this.dataList[i].price * this.dataList[i].num)
                }
            }
        } else {
            let pros = sessionStorage.getItem('buy')
            if (pros) {
                let pro = JSON.parse(pros)
                pro.num = 1
                this.totalPrice = pro.price
                this.dataList = [pro]
            }
        }
        let selAddr = this.$route.query.selAddr
        if (selAddr) {
            this.addrInfo = JSON.parse(sessionStorage.getItem('selAddr'))
            this.isAddr = true
        } else {
            this.getAddr()
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

function keepTwoDecimal (num) {
    var result = parseFloat(num)
    if (isNaN(result)) {
        return 0
    }
    result = Math.round(num * 100) / 100
    return result
}

</script>

<style lang="scss">
    .orderbooking-box{
        height: 100%;
        background-color: #f9f9f9;
        .addr-box{
            padding: .2rem .6rem;
            background: url('../assets/images/right.png') no-repeat center;
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
                background: url('../assets/images/site.png') no-repeat center;
                background-size: .24rem;
            }
            .addr-info{
                margin-top: .1rem;
                color: #777;
            }
            .newaddr{
                display: flex;
                line-height: .6rem;
                .btn{
                    width: .6rem;
                    height: .6rem;
                    background: url('../assets/images/jia.png') no-repeat center;
                    background-color: #f00;
                    background-size: .42rem;
                    color: #fff;
                    font-size: .64rem;
                    text-align: center;
                    margin-right: .2rem;
                    border-radius: .1rem;
                    box-sizing: border-box;
                }
            }
        }
        .products-box{
            margin-top: .2rem;
            .storename{
                height: .8rem;
                line-height: .8rem;
                color: #333;
                padding-left: .6rem;
                background: url('../assets/images/store.png') no-repeat center;
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
                        padding: .1rem 0 0 .2rem;
                        box-sizing: border-box;
                        .name{
                            font-weight: bold;
                        }
                        .attr{
                            height: .3rem;
                            color: #777;
                            margin: .1rem 0;
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
