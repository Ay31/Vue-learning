# Vue 动画

单元素/组件的过渡，即条件渲染（v-if），条件展示（v-show）,动态组件，组件根节点。

使用<transition name="fade"></transition>包裹，在css中通过类控制

fade-enter进入前生效，进入后第一帧移除，fade-enter-to，进入第一帧后生效，动画完成后移除，fade-enter-active,进入前生效，动画完成后移除。

fade-leave离开前生效，离开后第一帧移除，fade-leave-to,离开第一帧后生效，动画完成后移除，fade-leave-avtive，离开前生效，动画完成后移除。（删除是从有到无，离开前不能直接opacity0）

没有name属性可直接用v-enter类名。



css3 @keyframes 关键帧

``` css
    <style>
        @keyframes test {
            from {                   //等同于0%
                transform: scale(0);
            }
            50% {
                transform: scale(1.5);
            }
            to {                        //等同于100%
                transform: scale(1);
            }
        }
        .fade-enter-active {
            transform-origin: left center;
            animation: test 1s;

        }
        .fade-leave-active {
            transform-origin: left center;
            animation: test 1s reverse;
        }
    </style>
```



transition中的状态类名可以自定义<transition name="fade" enter-active-class = "animated swing"></transition>，因此可以通过这种办法使用css库



如果想同时调用过渡效果和动画，可以在定义类名中添加原来的类名，再于css代码中加入过渡效果，如果想刷新第一次时就有动画效果（初始渲染），可以添加`appear`特性，动画时间和过渡时间冲突时，可以利用type选择优先类型，也可以使用`:duration`属性分别设置进入和离开的持续时间。

``` html
<div id="root">
        <transition 
        name="fade" 
        :duration="{enter:3000,leave:5000}"   //type="transition"/"animation "
        appear
        enter-active-class="animated swing fade-enter-active" 
        leave-active-class="animated shake fade-leave-active"
        appear-active-class="animated swing"
        >
            <div v-if="show">Hello Vue!</div>
        </transition>
        <button @click="handleClick">click</button>
    </div>
```



使用js动画,使用钩子`@enter`之类，同理有`leave`对应的钩子

``` html
    <div id="root">
        <transition 
                    @before-enter="handleBeforeEnter"
                    @enter="handleEnter" 
                    @after-enter="handleAfterEnter">
            <div v-if="show">Hello Vue!</div>
        </transition>
        <button @click="handleClick">click</button>
    </div>
```

``` vue
 <script>
        new Vue({
            el: "#root",
            data: {
                show: true
            },
            methods: {
                handleClick: function () {
                    this.show = !this.show
                },
                handleBeforeEnter: function (el) {
                    el.style.color = "red";
                },
                handleEnter: function (el, done) {
                    setTimeout(() => {
                        el.style.background = "green";
                    }, 2000)
                    setTimeout(() => {
                        done()
                    }, 4000)
                },
                handleAfterEnter: function (el) {
                    el.style.background = "black"
                }
            }
        })
    </script>
```



使用js动画库

``` vue
                handleBeforeEnter: function (el) {
                    el.style.opacity = 0;
                },
                handleEnter: function (el, done) {
                    Velocity(el,{opacity:1},{duration:1000, complete: done},)
                },
                handleAfterEnter: function (el) {
                    el.style.background = "green";
                }
```



多元素过渡，div需要添加key值，否则vue会复用，mode控制进入和离开的先后顺序。

``` html
        <transition mode="out-in">
            <div v-if="show" key="hello">Hello Vue!</div>
            <div v-else key="bye">Bey Vue!</div>
        </transition>
```



使用动态组件过渡

``` html
    <transition mode="out-in">
            <component :is="type"></component>
        </transition>
        <button @click="handleClick">click</button>   //修改type值
```



给列表过渡,<transition-group>实际是给每一个<div>都添加了一层<transition>，所以加上css样式即可。

``` html
        <transition-group>
        <div v-for="item of list":key="item.id">{{item.text}}</div>
    </transition-group>
```



封装动画

``` html
        <fade :show="show">
            <h1 >
                HEELO WORLD
            </h1>
        </fade>
```

``` vue
        Vue.component('fade', {
            props: ['show'],
            template: `<transition>
                <slot v-if = "show"></slot>
                </transition>`
        })        //css代码另写
```

``` vue
Vue.component('fade', {
            props: ['show'],
            template: `<transition @before-enter="handleBeforeEnter"
                                    @enter="handleEnter">
                <slot v-if = "show"></slot>
                </transition>`,
            methods:{
                handleBeforeEnter: function(el){
                    el.style.color = "red";
                },
                handleEnter: function(el){
                    setTimeout(()=> 
                    el.style.color = "green"
                    ,2000)
                }
            }
        })                 //动画效果也封装入组件中
```

