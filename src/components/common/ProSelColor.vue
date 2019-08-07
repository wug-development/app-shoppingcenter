<template>
    <div id="ProSelColor" v-if="isDisplay" class="proselcolor-layer">
        <div class="proselcolor-box">
            <div class="proselcolor-proinfo">
                <div class="proselcolor-img">
                    <img :src="proinfo.selModel.color.productcolorImage" alt="">
                </div>
                <div class="proselcolor-name">
                    <div>{{proinfo.name}}</div>
                    <div class="proselcolor-price">&yen;{{proinfo.selModel.color.price}}</div>
                </div>
                <div class="proselcolor-close" @click="close"><span><img src="../../assets/images/close.png" alt=""></span></div>
            </div>
            <div class="proselcolor-procolor" v-if="colorArr.length">
                <div class="proselcolor-title">颜色：</div>
                <ul class="proselcolor-list">
                    <li v-for="(c, i) in colorArr" @click="checkColor(c, 'color')" :class="(c.productcolor==proinfo.selModel.color.productcolor?'cur':'') + (c.stock < 1?' disable':'')" :key="i">{{c.productcolorValue }}</li>
                </ul>
            </div>
            <div class="proselcolor-procolor" v-if="sizeArr.length">
                <div class="proselcolor-title">大小：</div>
                <ul class="proselcolor-list">
                    <li v-for="(c, i) in sizeArr" @click="checkColor(c, 'size')" :class="(c.productsize==proinfo.selModel.color.productsize?'cur':'') + (c.stock < 1?' disable':'')" :key="i">{{c.productsizeValue }}</li>
                </ul>
            </div>
            <div class="proselcolor-pronum">
                <div class="proselcolor-title">购买数量：</div>
                <div class="proselcolor-num">
                    <div class="num-jian" @click="jian"><img src="../../assets/images/jian.png" alt=""></div>
                    <div class="num-num"><input type="tel" v-model="num" /></div>
                    <div class="num-jia" @click="jia">+</div>
                </div>
            </div>
            <div class="proselcolor-btn">
                <div class="proselcolor-addcart" @click="addCart">加入购物车</div>
                <div class="proselcolor-buynow" @click="buyNow">立即购买</div>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    props: ['isShow', 'pro', 'maxNum'],
    name: 'ProSelColor',
    data () {
        return {
            isDisplay: false,
            proinfo: {},
            num: 1,
            mNum: 1,
            selcolor: {},
            colorArr: [],
            sizeArr: []
        }
    },
    methods: {
        close: function () {
            this.isDisplay = false
            this.$emit('close', false)
        },
        jia: function () {
            if (this.num < this.proinfo.selModel.color.stock) {
                this.num = this.num + 1
            } else {
                this.Toast('该商品库存不足最多可买数量' + this.proinfo.selModel.color.stock)
            }
        },
        jian: function () {
            if (this.num > 1) {
                this.num = this.num - 1
            }
        },
        addCart: function () {
            var s = sessionStorage.getItem('cart')
            var pros = []
            if (s) {
                pros = JSON.parse(s)
                var p = pros.find((e) => { return e.id === this.proinfo.id && e.selModel.color.id === this.proinfo.selModel.color.id })
                if (p) {
                    p.num = this.num + p.num
                } else {
                    this.proinfo.num = this.num
                    this.proinfo.issel = true
                    pros.push(this.proinfo)
                }
                sessionStorage.setItem('cart', JSON.stringify(pros))
                this.Toast('加入成功！')
            } else {
                this.proinfo.num = this.num
                this.proinfo.issel = true
                pros.push(this.proinfo)
                sessionStorage.setItem('cart', JSON.stringify(pros))
                this.Toast('加入成功！')
            }
            this.close()
        },
        buyNow: function () {
            this.close()
            sessionStorage.setItem('buy', JSON.stringify(this.proinfo))
            this.$router.push({
                path: '/orderbooking?ty=buy'
            })
        },
        checkColor (c, t) {
            if (c.stock > 0) {
                this.proinfo.selModel.color = c
                let arr = this.proinfo.productColorSizeStocks
                if (t === 'color') {
                    let arrs = []
                    arr.forEach(item => {
                        let _bindex = arrs.findIndex(b => {
                            if (c.productcolor === item.productcolor) {
                                return b.productsize === item.productsize
                            } else {
                                return true
                            }
                        })
                        if (_bindex < 0) {
                            arrs.push(item)
                        }
                    })
                    this.sizeArr = arrs
                } else {
                    let arrc = []
                    arr.forEach(item => {
                        let _aindex = arrc.findIndex(a => {
                            if (c.productsize === item.productsize) {
                                return a.productcolor === item.productcolor
                            } else {
                                return true
                            }
                        })
                        if (_aindex < 0) {
                            arrc.push(item)
                        }
                    })
                    this.colorArr = arrc
                }
            }
        }
    },
    watch: {
        isShow: function (v) {
            this.isDisplay = v
        },
        pro: function (v) {
            this.proinfo = v
            let arr = v.productColorSizeStocks
            let arrc = []
            let arrs = []
            if (arr) {
                arr.forEach(item => {
                    let _aindex = arrc.findIndex(a => {
                        if (this.proinfo.selModel.color.productsize === item.productsize) {
                            return a.productcolor === item.productcolor
                        } else {
                            return true
                        }
                    })
                    if (_aindex < 0) {
                        arrc.push(item)
                    }
                    let _bindex = arrs.findIndex(b => {
                        if (this.proinfo.selModel.color.productcolor === item.productcolor) {
                            return b.productsize === item.productsize
                        } else {
                            return true
                        }
                    })
                    if (_bindex < 0) {
                        arrs.push(item)
                    }
                })
            }
            this.sizeArr = arrs
            this.colorArr = arrc
        },
        maxNum: function (v) {
            this.mNum = v
        }
    }
}
</script>

<style lang="scss">
    .proselcolor-layer{
        position: fixed;
        width: 100%;
        height: 100%;
        background-color: rgba(0, 0, 0, .7);
        left: 0;
        top: 0;
        z-index: 11;
        .proselcolor-box{
            position: absolute;
            left: 0;
            bottom: 0;
            width: 100%;
            background-color: #FFF;
            padding-left: .24rem;
            padding-bottom: .9rem;
            box-sizing: border-box;
            .proselcolor-title{
                font-size: .28rem;
                height: .8rem;
                line-height: .86rem;
            }
            .proselcolor-proinfo{
                display: flex;
                position: relative;
                height: 1.4rem;
                border-bottom: .02rem solid #ddd;
                justify-content: space-between;
                .proselcolor-img{
                    border: .02rem solid #ddd;
                    width: 1.4rem;
                    height: 1.4rem;
                    border-radius: .08rem;
                    background-color: #fff;
                    box-sizing: border-box;
                    position: relative;
                    top: -.2rem;
                    img{
                        display: block;
                        width: 100%;
                    }
                }
                .proselcolor-name{
                    width: 5.16rem;
                    padding: .16rem .16rem;
                    font-size: .28rem;
                    box-sizing: border-box;
                    .proselcolor-price{
                        margin-top: .08rem;
                        color: #f00000;
                    }
                }
                .proselcolor-close{
                    width: .7rem;
                    span{
                        display: block;
                        width: 20px;
                        height: 20px;
                        border-radius: 20px;
                        border: .02rem solid #999;
                        margin-top: .14rem;
                        img{
                            display: block;
                            width: 12px;
                            margin: 4px auto;
                        }
                    }
                }
            }
            .proselcolor-procolor{
                border-bottom: .02rem solid #ddd;
                .proselcolor-list{
                    display: flex;
                    flex-wrap: wrap;
                    li{
                        font-size: .24rem;
                        margin: 0 .2rem .2rem 0;
                        border: .02rem solid #999;
                        line-height: .5rem;
                        height: .5rem;
                        padding: 0 .25rem;
                        border-radius: .5rem;
                        img{
                            display: block;
                            width: 100%;
                        }
                    }
                    .cur{
                        color: #f00000;
                        border: .02rem solid #f00000;
                        background-color: #FFDADA;
                    }
                    .disable{
                        color: #ccc;
                        border: .02rem solid #ccc;
                    }
                }
            }
            .proselcolor-pronum{
                display: flex;
                justify-content: space-between;
                padding-top: .2rem;
                .proselcolor-title{
                    line-height: .4rem;
                }
                .proselcolor-num{
                    display: flex;
                    padding-right: .4rem;
                    line-height: .6rem;
                    .num-jian{
                        width: .8rem;
                        height: .6rem;
                        border: .02rem solid #ccc;
                        box-sizing: border-box;
                        text-align: center;
                        img{
                            margin-top: .16rem;
                            width: .24rem;
                        }
                    }
                    .num-num{
                        width: .6rem;
                        height: .6rem;
                        border-top: .02rem solid #ccc;
                        border-bottom: .02rem solid #ccc;
                        box-sizing: border-box;
                        text-align: center;
                        font-size: .4rem;
                        input{
                            display: block;
                            width: 100%;
                            height: 100%;
                            border: 0;
                            text-align: center;
                            font-size: .28rem;
                            padding: .2rem 0;
                            box-sizing: border-box;
                        }
                    }
                    .num-jia{
                        width: .8rem;
                        height: .6rem;
                        border: .02rem solid #ccc;
                        box-sizing: border-box;
                        text-align: center;
                        font-size: .4rem;
                    }
                }
            }
            .proselcolor-btn{
                position: absolute;
                left: 0;
                bottom: 0;
                width: 100%;
                display: flex;
                text-align: center;
                height: .9rem;
                line-height: .9rem;
                font-size: .28rem;
                .proselcolor-addcart{
                    width: 50%;
                    color: #fff;
                    background-color: #414141;
                }
                .proselcolor-buynow{
                    width: 50%;
                    color: #fff;
                    background-color: #c71f2c;
                }
            }
        }
    }
</style>
