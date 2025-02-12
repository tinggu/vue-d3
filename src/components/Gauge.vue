<template>
    <div :class="gaugeArea" ref="area">
        <div :style="cssGauge">
            <div :style="cssDotWrap">
                <div :style="Object.assign({},{ transform: 'rotateZ('+i*4+'deg)', },cssDot)"
                     v-for="(dot,i) in 90"
                     :key="i"
                ></div>
            </div>
            <div :style="cssDotWrap2">
                <div :class="gaugeBack"></div>
                <div :style="Object.assign({},{ transform: 'rotateZ('+i*3+'deg)', background:setScaleColor(i) },cssDot2)"
                     v-for="(dot,i) in 120"
                     :key="i"
                ></div>
            </div>
            <div :style="Object.assign({},{  transform: 'rotateZ('+i*360 /12+'deg)', })"
                 v-for="(dot,i) in 12"
                 :class="gaugeScaleVal"
                 :key="i"
            >
                {{getScaleValue(i)}}
            </div>
            <div :style="Object.assign({},{ transform: 'rotateZ('+getGaugeRotate+'deg)'  })"
                 :class="gaugeNeedle"
            ></div>
            <div :class="gaugeCircle"></div>
            <div :class="gaugeDot"></div>
        </div>
        <div :class="gaugeTitle">泄漏指数：{{value}}</div>
    </div>
</template>

<script>
    export default {
        name: "Gauge",
        data () {
            return {
                prefixCss: 'jhc-', // 所有 CSS 的前缀
                value: 33, // 当前默认value
                max: 100, // 当前刻度最大值
                outerScale: 0.85, // 表盘整体占整个 div 元素的比例
                innerScale: 0.75, // 内层刻度占外层的百分比
                cssGauge: {// 整个仪表盘的盒子
                    position: 'absolute',
                    width: '100%',
                    height: '100%',
                    borderRadius: '50%'
                },
                cssDotWrap: {// 装外层装饰刻度的盒子
                    width: '100%',
                    height: '100%',
                    position: 'absolute',
                    transform: 'translateX(250px)'
                },
                cssDotWrap2: {// 装内层指示刻度的盒子
                    width: '100%',
                    height: '100%',
                    position: 'absolute',
                    transform: 'translateX(120px)'
                },
                cssDot: {// 外层装饰刻度们
                    position: 'absolute',
                    backgroundColor: '#509fef',
                    transformOrigin: '0px 165px'
                },
                cssDot2: {// 内层指示刻度们
                    position: 'absolute',
                    backgroundColor: '#509fef',
                    transformOrigin: '0px 165px'
                },
                colorList: [], // 内层刻度的颜色
                timer: null, // 刷新图表的timer
                style: { bottom: '1px' }// 仪表盘标题的style
            }
        },
        mounted () {
            // 定时修改 value 值
            this.timer = setInterval(() => {
                this.value = Math.ceil((Math.random() * 100))
            }, 2 * 1000)
            // 获取内层指示刻度的颜色
            this.getColorList()
            // 设置仪表盘图的 size
            this.setSize()
        },
        beforeDestroy () {
            clearInterval(this.timer)
        },
        methods: {
            // 设置内环刻度颜色
            setScaleColor (i) {
                if (i < 50) {
                    return this.colorList[i + 70]
                } else {
                    return this.colorList[i - 50]
                }
            },
            // 获取颜色列表：由蓝色逐渐变红
            getColorList () {
                var colorList = []
                var red = {
                    max: 173,
                    min: 20
                }
                var green = {
                    max: 146,
                    min: 25
                }
                var blue = {
                    max: 216,
                    min: 45
                }
                var lth = 120
                for (var i = 0; i < lth; i++) {
                    var color = `rgb(${red.max - (red.max - red.min) / lth * i},${green.min + (green.max - green.min) / lth * i},${blue.min + (blue.max - blue.min) / lth * i})`
                    colorList.unshift(color)
                }
                this.colorList = colorList
            },
            // 设置仪表盘图的大小
            setSize () {
                let width = this.getWidth() // 获取 div 的宽度
                let height = this.getHeight() // 获取 div 的高度
                let shortLth = width > height ? height : width // 获取较短边
                let length = shortLth * this.outerScale // 根据较短边 以及 外层比例，计算实际使用的长度
                let paddingW = (width - length) / 2 // 获取距离左部的距离
                let paddingH = (height - length) / 2 // 获取距离顶部的距离
                // 设置仪表盘整体的大小形状
                this.setStates('cssGauge', {
                    height: length + 'px',
                    width: length + 'px',
                    top: paddingH + 'px',
                    left: paddingW + 'px'
                })
                let dotRadius = length * 0.5 // 设置外层刻度的半径
                let innerScale = this.innerScale // 内层刻度占外层的比例
                // 刻度的高度和宽度
                let dotHeight = length * 0.03 + 'px'
                let dotWidth = length * 0.01 + 'px'
                // 外层刻度的样式
                this.setStates('cssDot', {
                    transformOrigin: `${0}px ${dotRadius}px`,
                    height: dotHeight,
                    width: dotWidth
                })
                // 内层刻度的样式
                this.setStates('cssDot2', {
                    transformOrigin: `${0}px ${dotRadius * innerScale}px`,
                    height: dotHeight,
                    width: dotWidth
                })
                // 设置外层刻度所在区域的位置
                this.setStates('cssDotWrap', {
                    transform: `translateX(${length * 0.5}px)`
                })
                // 设置内刻度所在区域的位置
                this.setStates('cssDotWrap2', {
                    transform: `translate(${length * 0.5}px,${length * (1 - innerScale) / 2}px)`
                })
            },
            // 获取表盘的刻度值
            getScaleValue (i) {
                if (i !== 6) {
                    return (i >= 7 ? ((i - 7) / 10) : ((i + 5) / 10)) * this.max
                } else {
                    return ''
                }
            },
            getWidth () { // 获取指定容器的宽度
                return this.getRef('area').offsetWidth || 200
            },
            getHeight () { // 获取指定容器的高度
                return this.getRef('area').offsetHeight || 200
            },
            getRef (ref) { // 获取指定 ref 对象
                // eslint-disable-next-line no-mixed-operators
                return this.$refs && this.$refs[ref] || {}
            },
            // 模仿 react 的states
            setStates (prop, data) {
                let cache = this[prop]
                this[prop] = Object.assign({}, cache, JSON.parse(JSON.stringify(data)))
            }
        },
        computed: {
            gaugeArea () { // 仪表盘的区域的class
                return `${this.prefixCss}gauge-area`
            },
            gaugeCircle () { // 仪表盘的圆环的class
                return `${this.prefixCss}gauge-circle`
            },
            gaugeDot () { // 仪表盘的中心点的class
                return `${this.prefixCss}gauge-center`
            },
            gaugeBack () { // 仪表盘的背景图的class
                return `${this.prefixCss}gauge-back`
            },
            gaugeNeedle () { // 仪表盘的指针的class
                return `${this.prefixCss}gauge-needle`
            },
            gaugeScaleVal () { // 仪表盘刻度的读数的class
                return `${this.prefixCss}gauge-scale`
            },
            getGaugeRotate () { // 获取仪表盘的角度
                return (this.value / this.max * 100) * (360 / 120) - 150
            },
            gaugeTitle () { // 仪表盘的title的class
                return `${this.prefixCss}gauge-title`
            }
        }
    }
</script>

<style scoped>
    /**仪表盘的css开始**/
    /* 仪表盘区域 */
    .jhc-gauge-area {
        width: 500px;
        height: 500px;
        position: relative;
        background: #050842;
    }

    /* 仪表盘中圆形的环 */
    .jhc-gauge-circle {
        width: 88%;
        height: 88%;
        border: 2px solid #509fef;
        border-radius: 50%;
        position: absolute;
        top: 5%;
        left: 6%;
    }

    /* 仪表盘的圆心 */
    .jhc-gauge-center {
        width: 8%;
        height: 8%;
        border-radius: 50%;
        position: absolute;
        background-color: #509fef;
        z-index: 20;
        left: 46%;
        top: 46%;
        box-shadow: 0px 0px 10px 1px #000;
    }

    /* 仪表盘的背景图片 */
    .jhc-gauge-back {
        width: 66%;
        height: 66%;
        border-radius: 50%;
        position: absolute;
        top: 4%;
        left: -33%;
        background: url(https://jhcan333.github.io/can-Share/image/clock/gaugeBack.png) no-repeat;
        background-size: 95% 95%;
        background-position: center center;
    }

    /* 仪表盘的指针 */
    .jhc-gauge-needle {
        z-index: 5;
        width: 8%;
        height: 26%;
        position: absolute;
        left: 46%;
        bottom: 50%;
        transition: all 0.5s ease-in-out;
        transform-origin: center bottom;
        background: url(https://jhcan333.github.io/can-Share/image/clock/hourPoint.png) no-repeat;
        background-size: 100% 100%;
    }

    /* 仪表盘的标题 */
    .jhc-gauge-title {
        color: #fff;
        position: absolute;
        width: 100%;
        text-align: center;
        font-weight: bold;
        bottom: 3px;
    }

    /* 仪表盘的刻度 */
    .jhc-gauge-scale {
        color: #fff;
        z-index: 2;
        text-align: center;
        width: 10%;
        height: 43%;
        position: absolute;
        left: 45%;
        bottom: 50%;
        transform-origin: center bottom;
    }

    /**仪表盘的css结束**/
</style>
