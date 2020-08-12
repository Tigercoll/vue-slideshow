---
title: vue实现轮播图
categories: vue
tag: [vue,轮播图]
typora-root-url: ..

---

## 前言

最近项目需要移动端实现轮播图,查看文档,总结一下

## 原理

![image-20200810235057262](../images/vue%E5%AE%9E%E7%8E%B0%E8%BD%AE%E6%92%AD%E5%9B%BE/image-20200810235057262.png)

轮播 其实就是让图片排在一排,等鼠标或手指让其显示在视口,可被看见的区域内.

## 制作轮播图

### 搭建基本dom结构

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>轮播图demo</title>
    <style>
        * {
            margin: 0;
            padding: 0;
        }
        
        .swiper {
            /* 定义视口 */
            height: 200px;
            width: 300px;
            margin: 100px auto;
            border: 1px solid red;
            /* 超出部分隐藏 */
            overflow: hidden;
        }
        
        .slides {
            display: flex;
            height: 100%;
        }
        
        .slides li {
            /* 去掉前面的黑点 */
            list-style: none;
        }
        
        li img {
            height: 100%;
        }
    </style>
</head>

<body>
    <div id="app">
        <div class="swiper">
            <ul class="slides">
                <li :key="index" v-for="(item,index) in items">
                    <img :src="item.url">
                </li>
            </ul>
        </div>
    </div>

    <script src="js/vue.js"></script>
    <script>
        new Vue({
            el: '#app',
            data() {
                return {
                    items: [{
                        id: 1,
                        url: '/imgs/1.jpg'
                    }, {
                        id: 2,
                        url: '/imgs/2.jpg'
                    }, {
                        id: 3,
                        url: '/imgs/3.jpg'
                    }, {
                        id: 4,
                        url: '/imgs/4.jpg'
                    }, ]
                }
            },
        })
    </script>
</body>

</html>
```

![image-20200812185854004](/images/vue%E5%AE%9E%E7%8E%B0%E8%BD%AE%E6%92%AD%E5%9B%BE/image-20200812185854004.png)

### 实现简单的轮播

为了让每张图都填满整个视口,需要设置每个图片的宽度为视口宽度,整个的宽度就是所有图片宽度的总和

```vue
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>轮播图demo</title>
    <style>
        * {
            margin: 0;
            padding: 0;
        }
        
        .swiper {
            /* 定义视口 */
            height: 200px;
            width: 300px;
            margin: 100px auto;
            border: 1px solid red;
            /* 超出部分隐藏 */
            overflow: hidden;
        }
        
        .slides {
            display: flex;
            height: 100%;
        }
        
        .slides li {
            width: 100%;
            /* 去掉前面的黑点 */
            list-style: none;
        }
        
        li img {
            width: 100%;
            height: 100%;
        }
    </style>
</head>

<body>
    <div id="app">
        <div class="swiper" ref="viewPortRef">
            <ul class="slides" ref="box" :style="translateStyle">
                <li :key="index" v-for="(item,index) in items">
                    <img :src="item.url">
                </li>
            </ul>
        </div>
    </div>

    <script src="js/vue.js"></script>
    <script>
        new Vue({
            el: '#app',
            data() {
                return {
                    // 定时器
                    timer: null,
                    // 当前index
                    currentIndex: 0,
                    // 视口
                    viewport: 0,
                    items: [{
                        id: 1,
                        url: '/imgs/1.jpg'
                    }, {
                        id: 2,
                        url: '/imgs/2.jpg'
                    }, {
                        id: 3,
                        url: '/imgs/3.jpg'
                    }, {
                        id: 4,
                        url: '/imgs/4.jpg'
                    }, ]
                }
            },
            mounted() {
                // 初始化宽度
                this.init()
                    // 自动轮播
                this.autoplay()
            },
            methods: {
                // 初始化
                init() {
                    // 获取视口的宽度
                    this.viewport = this.$refs.viewPortRef.clientWidth
                        // 计算图片外层盒子的大小
                    this.$refs.box.style.width = this.items.length * this.viewport + 'px'
                },
                // 自动轮播 ,其实就是通过transition 偏移X
                autoplay() {
                    this.timer = setInterval(() => {
                        this.currentIndex++
                            if (this.currentIndex === this.items.length) {
                                this.currentIndex = 0
                            }

                    }, 2000)
                }
            },
            // 通过计算属性来获取偏移的X
            computed: {
                translateStyle() {
                    const translate = `translateX(-${this.currentIndex*this.viewport}px)`
                    return {
                        transform: translate
                    }
                }
            },
        })
    </script>
</body>

</html>
```

![GIF 2020-8-12 19-23-23](/images/vue%E5%AE%9E%E7%8E%B0%E8%BD%AE%E6%92%AD%E5%9B%BE/GIF%202020-8-12%2019-23-23.gif)

简单实现了轮播的效果,不过有点粗糙,我们加入过渡

### 加入过渡

单单在样式中加入过渡,我们会发现轮播到最后一张图片的时候,会转到第一张

![GIF 2020-8-12 19-27-02](/images/vue%E5%AE%9E%E7%8E%B0%E8%BD%AE%E6%92%AD%E5%9B%BE/GIF%202020-8-12%2019-27-02.gif)

这样的效果,我们并不希望得到,我们希望他是无缝的,

修改

```js
     autoplay() {
         this.timer = setInterval(() => {
         this.isTransition = true
         this.currentIndex++
         if (this.currentIndex === this.items.length) {
            this.isTransition = false
            this.currentIndex = 0
              }
      }, 2000)
 }
```

![GIF 2020-8-12 19-37-15](/images/vue%E5%AE%9E%E7%8E%B0%E8%BD%AE%E6%92%AD%E5%9B%BE/GIF%202020-8-12%2019-37-15.gif)

从最后一张跳到第一张还是有点不舒服,那么我们怎么办呢?

我们需要重新定义一个列表

![image-20200812195232370](/images/vue%E5%AE%9E%E7%8E%B0%E8%BD%AE%E6%92%AD%E5%9B%BE/image-20200812195232370.png)

然后滚动到4的时候再到1然后再跳转到前面的1(当然最前面的4是为我们做滑动的时候准备的)

```vue
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>轮播图demo</title>
    <style>
        * {
            margin: 0;
            padding: 0;
        }
        
        .swiper {
            /* 定义视口 */
            height: 200px;
            width: 300px;
            margin: 100px auto;
            border: 1px solid red;
            /* 超出部分隐藏 */
            overflow: hidden;
        }
        
        .slides {
            display: flex;
            height: 100%;
            /* 加入过渡 */
        }
        
        .slides li {
            width: 100%;
            /* 去掉前面的黑点 */
            list-style: none;
        }
        
        li img {
            width: 100%;
            height: 100%;
        }
    </style>
</head>

<body>
    <div id="app">
        <div class="swiper" ref="viewPortRef">
            <ul class="slides" ref="box" :style="translateStyle">
                <li :key="index" v-for="(item,index) in newList">
                    <img :src="item.url">
                </li>
            </ul>
        </div>
    </div>

    <script src="js/vue.js"></script>
    <script>
        new Vue({
            el: '#app',
            data() {
                return {
                    newList: [],
                    // 定时器
                    timer: null,
                    // 当前index
                    currentIndex: 0,
                    // 视口
                    viewport: 0,
                    isTransition: false,
                    items: [{
                        id: 1,
                        url: '/imgs/1.jpg'
                    }, {
                        id: 2,
                        url: '/imgs/2.jpg'
                    }, {
                        id: 3,
                        url: '/imgs/3.jpg'
                    }, {
                        id: 4,
                        url: '/imgs/4.jpg'
                    }, ]
                }
            },
            mounted() {
                // 初始化宽度
                this.init()
                    // 自动轮播
                this.autoplay()
            },
            methods: {
                // 初始化
                init() {
                    const first = this.items.slice(0, 1)
                    const last = this.items.slice(-1)
                    this.newList = [...last, ...this.items, ...first]
                        // 获取视口的宽度
                    this.viewport = this.$refs.viewPortRef.clientWidth
                        // 计算图片外层盒子的大小
                    this.$refs.box.style.width = this.newList.length * this.viewport + 'px'
                },
                // 自动轮播 ,其实就是通过transition 偏移X
                autoplay() {
                    this.timer = setInterval(() => {
                        // 加上延时,造成视觉误差
                        setTimeout(() => {
                            this.isTransition = true
                            this.currentIndex++
                        }, 40)

                        if (this.currentIndex === this.newList.length - 1) {
                            this.isTransition = false
                            this.currentIndex = 1
                        }


                    }, 2000)
                }
            },
            // 通过计算属性来获取偏移的X
            computed: {
                translateStyle() {
                    const translate = `translateX(-${this.currentIndex*this.viewport}px)`
                    const transition = this.isTransition ? 'all 1s' : 'none'
                    return {
                        transform: translate,
                        transition: transition
                    }
                }
            },
        })
    </script>
</body>

</html>
```

![GIF 2020-8-12 19-58-16](/images/vue%E5%AE%9E%E7%8E%B0%E8%BD%AE%E6%92%AD%E5%9B%BE/GIF%202020-8-12%2019-58-16.gif)

### 鼠标滑动

滑动有三个事件:

` touchstart` 按住时,`touchmove` 移动 ,`touchend` 结束 (当然这三个事件只在移动端有效)

pc端需要`mousedown mousemove mouseup` 三个事件,这里我用到的是移动端

```js
     onTouchStart(e) {
                    // 关闭自动播放
                    this.stopplay()
                        // 关闭过渡
                    this.isTransition = false
                        // 当前是0的时候,是最后第二个
                    if (this.currentIndex === 0) {
                        this.currentIndex = this.newList.length - 2
                    }
                    if (this.currentIndex === this.newList.length - 1) {
                        this.currentIndex = 1
                    }
                    // 获取按住的X坐标位置
                    this.toucheStartX = e.targetTouches[0].clientX;
                    console.log(this.toucheStartX);
                },
                onTouchMove(e) {

                    // 获取移动的偏移量
                    this.offsetX = this.toucheStartX - e.targetTouches[0].clientX;

                },
                onTouchEnd() {
                    // 开启效果
                    this.isTransition = true;
                    // 四舍五入
                    let currentIndex = Math.round(this.offsetX / this.viewport)
                    this.currentIndex = currentIndex + this.currentIndex
                        // 重置offsetX
                    this.offsetX = 0
                        // 自动播放
                    this.autoplay()
                }
```

![GIF 2020-8-12 21-17-02](/C:/Users/Tiger/Desktop/GifCam/GIF%202020-8-12%2021-17-02.gif)

### 底部 小圆点

#### 添加dom

```html
<div class="swiper" ref="viewPortRef">
            <ul class="slides" ref="box" :style="translateStyle" @touchstart="onTouchStart" @touchmove="onTouchMove" @touchend="onTouchEnd">
                <li :key="index" v-for="(item,index) in newList">
                    <img :src="item.url">
                </li>
            </ul>
            <div class="sliderListDot">
                <span class="sliderDot" :key="index" v-for="(item,index) in items" :class="{active : activeIndex==index}"></span>
            </div>
        </div>
```

#### 设计样式

```css
  .sliderListDot {
            position: absolute;
            left: 50%;
            bottom: 20px;
            display: flex;
            transform: translateX(-50%);
        }
        
        .sliderDot {
            width: 14px;
            height: 14px;
            background-color: #fff;
            margin: 0 10px;
            border-radius: 50%;
            opacity: 0.6;
            transition: all 0.5s;
        }
        
        .active {
            width: 36px;
            background-color: #fff;
            border-radius: 7px;
            opacity: 1;
        }
```

#### 添加计算属性

```js
            activeIndex() {
                    let index = 0
                        // 当currentIndex=0的时候就是列表的最后一个
                    if (this.currentIndex === 0) {
                        index = this.items.length-1

                    }
                    // 当是倒数第二个的时候就是第0个 
                    else if (this.currentIndex === this.newList.length - 1) {
                        index = 0

                    }
                    // 其余都是-1 
                    else {
                        index = this.currentIndex - 1
                    }
                    return index
                }
```

![GIF 2020-8-12 21-52-25](/images/vue%E5%AE%9E%E7%8E%B0%E8%BD%AE%E6%92%AD%E5%9B%BE/GIF%202020-8-12%2021-52-25.gif)

#### 鼠标移动到小圆点改变轮播

有两个事件:`mouseenter` 跟`mouseleave`

```js
 // 鼠标进入的时候
                enter(index) {

                    // 停止自动播放
                    this.stopplay()
                    this.currentIndex = index + 1
                },
                // 鼠标离开的时候
                leave() {

                    this.autoplay()
                }
```

## 完整代码

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>轮播图demo</title>
    <style>
        * {
            margin: 0;
            padding: 0;
        }
        
        .swiper {
            /* 定义视口 */
            height: 200px;
            width: 300px;
            margin: 100px auto;
            border: 1px solid red;
            /* 超出部分隐藏 */
            overflow: hidden;
            position: relative;
        }
        
        .slides {
            display: flex;
            height: 100%;
            /* 加入过渡 */
        }
        
        .slides li {
            width: 100%;
            /* 去掉前面的黑点 */
            list-style: none;
        }
        
        li img {
            width: 100%;
            height: 100%;
        }
        
        .sliderListDot {
            position: absolute;
            left: 50%;
            bottom: 20px;
            display: flex;
            transform: translateX(-50%);
        }
        
        .sliderDot {
            width: 14px;
            height: 14px;
            background-color: #fff;
            margin: 0 10px;
            border-radius: 50%;
            opacity: 0.6;
            transition: all 0.5s;
        }
        
        .active {
            width: 36px;
            background-color: #fff;
            border-radius: 7px;
            opacity: 1;
        }
    </style>
</head>

<body>
    <div id="app">
        <div class="swiper" ref="viewPortRef">
            <ul class="slides" ref="box" :style="translateStyle" @touchstart="onTouchStart" @touchmove="onTouchMove" @touchend="onTouchEnd">
                <li :key="index" v-for="(item,index) in newList">
                    <img :src="item.url">
                </li>
            </ul>
            <div class="sliderListDot">
                <span class="sliderDot" :key="index" v-for="(item,index) in items" :class="{active : activeIndex==index}" @mouseenter="enter(index)" @mouselevae="leave"></span>
            </div>
        </div>
    </div>

    <script src="js/vue.js"></script>
    <script>
        new Vue({
            el: '#app',
            data() {
                return {
                    // 获取按住时的X值
                    toucheStartX: 0,
                    newList: [],
                    // 定时器
                    timer: null,
                    // 当前index
                    currentIndex: 1,
                    // 偏移量
                    offsetX: 0,
                    // 视口
                    viewport: 0,
                    isTransition: false,
                    items: [{
                        id: 1,
                        url: '/imgs/1.jpg'
                    }, {
                        id: 2,
                        url: '/imgs/2.jpg'
                    }, {
                        id: 3,
                        url: '/imgs/3.jpg'
                    }, {
                        id: 4,
                        url: '/imgs/4.jpg'
                    }, ]
                }
            },
            mounted() {
                // 初始化宽度
                this.init()
                    // 自动轮播
                this.autoplay()
            },
            methods: {
                // 初始化
                init() {
                    const first = this.items.slice(0, 1)
                    const last = this.items.slice(-1)
                    this.newList = [...last, ...this.items, ...first]
                        // 获取视口的宽度
                    this.viewport = this.$refs.viewPortRef.clientWidth
                        // 计算图片外层盒子的大小
                    this.$refs.box.style.width = this.newList.length * this.viewport + 'px'
                },
                // 自动轮播 ,其实就是通过transition 偏移X
                autoplay() {
                    this.timer = setInterval(() => {
                        // 加上延时,造成视觉误差
                        setTimeout(() => {
                            this.isTransition = true
                            this.currentIndex++
                        }, 40)

                        if (this.currentIndex === this.newList.length - 1) {
                            this.isTransition = false
                            this.currentIndex = 1
                        }


                    }, 2000)
                },
                stopplay() {
                    this.timer && clearInterval(this.timer)
                },
                onTouchStart(e) {
                    // 关闭自动播放
                    this.stopplay()
                        // 关闭过渡
                    this.isTransition = false
                        // 当前是0的时候,是最后第二个
                    if (this.currentIndex === 0) {
                        this.currentIndex = this.newList.length - 2
                    }
                    if (this.currentIndex === this.newList.length - 1) {
                        this.currentIndex = 1
                    }
                    // 获取按住的X坐标位置
                    this.toucheStartX = e.targetTouches[0].clientX;
                    console.log(this.toucheStartX);
                },
                onTouchMove(e) {

                    // 获取移动的偏移量
                    this.offsetX = this.toucheStartX - e.targetTouches[0].clientX;

                },
                onTouchEnd() {
                    // 开启效果
                    this.isTransition = true;
                    // 四舍五入
                    let currentIndex = Math.round(this.offsetX / this.viewport)
                    this.currentIndex = currentIndex + this.currentIndex
                        // 重置offsetX
                    this.offsetX = 0
                        // 自动播放
                    this.autoplay()
                },
                // 鼠标进入的时候
                enter(index) {

                    // 停止自动播放
                    this.stopplay()
                    this.currentIndex = index + 1
                },
                // 鼠标离开的时候
                leave() {

                    this.autoplay()
                }
            },
            // 通过计算属性来获取偏移的X
            computed: {
                translateStyle() {
                    const translate = `translateX(-${this.currentIndex*this.viewport+this.offsetX }px)`
                    const transition = this.isTransition ? 'all 1s' : 'none'
                    return {
                        transform: translate,
                        transition: transition
                    }
                },
                activeIndex() {
                    let index = 0
                        // 当currentIndex=0的时候就是列表的最后一个
                    if (this.currentIndex === 0) {
                        index = this.items.length - 1
                    }
                    // 当是倒数第二个的时候就是第0个 
                    else if (this.currentIndex === this.newList.length - 1) {
                        index = 0
                    }
                    // 其余都是-1 
                    else {
                        index = this.currentIndex - 1
                    }
                    return index
                }
            },
        })
    </script>
</body>

</html>
```

