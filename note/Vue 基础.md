# Vue 基础2

计算属性，侦听器，方法

方法在每次渲染时都会重新加载，而计算属性和侦听器都是在监听的属性发生变化时再开始执行，在计算属性和侦听器都可用的情况下，使用计算属性，因为代码量少。



计算属性中，默认的return是get方法方法，但也可以手动添加set方法



class和style的绑定，vue中使用v-bind绑定style和class时，不仅可以使用字符串，还可以使用数组和对象。

class若绑定的是对象，则判断对象键值的true/flase（如果值是用data数据表示，若没有设置具体布尔值，默认存在则为true），绑定为数组时，则绑定的data数据键值的属性值，可绑定多个。

style若绑定时对象，则类似css写法，键值（样式名），属性（数据），如果遇到类似font-size这种写法，需要用驼峰写法代替（fontSize），绑定数组则可以在数组中添加多个对象。



v-if后可以跟v-if-else（v-if-else=“show==='b'”）和v-else（三者需要在连续行）。



vue为提高性能，会复用标签，如果想标签独立需要加入不同的key值，key不推荐使用index，应该使用id。



v-for列表渲染中，如果渲染的是数组则有（item，index），如果是对象则有（item，键值，数据）。



如果想要修改数组，有三种方法，1是使用变异方法（push，pull，shift，unshift，reserve，sort，splice），2是替换数组（因为数组属于对象的一种，是引用类型），使用set方法（Vue.set/vm.$set），Vue.set（vm.xxxlist,索引，添加/替换内容）。



修改对象则只有两种方法，一种是替换对象，一种是使用set方法，Vue.set（vm.xxxObject,键值，数据）。



对于某些标签（table，ul，ol，select等），里面包含的标签是有要求的，如table后是tbody然后是tr，此时如果想用组件替代tr，可以使用`is`，<tr is="component"></tr>,



子组件中，data必须是function，因为子组件可能被调用多次，每个子组件都应该有独立的data，而根实例只会被调用一次，所以可以是对象，function中return的属性实例的属性。



vue中想要操控dom元素，需要给元素加入ref属性进行注册，在普通DOM元素中加入ref="注册名"，引用的就是元素，this.$refs.xxx.innerHTML可以获取内容。在子组件中加入ref="注册名"，引用的将是子组件的实例，所以可以获取里面的属性，this.$refs.xxx.number。



vue中有单向数据流，即父组件可以向下传递数据，但子组件不能修改父组件的数据，所以子组件接收到父组件的数据后，可以在data中将其赋值给本组件定义的属性，然后使用自身的属性进行运算。



子组件中，可以通过事件触发向父组件传递值。

组件参数校验，在组件的props属性中，可以对传进的参数进行校验。

``` vue
props:["content","message"]   // 只接收
props:{
	content:String,        // 希望content是String类型
	message:[String,Number]   // mes可以是S或者N类型
}
props:{
	content:[String,Number],  //同上
	message:{
		type:String,     // 希望类型是S
		require:false,     //该参数是否是必须的
		default:'default value'  //，如果未传入参数，使用该默认值
		validator:function(value){   //对传入字符串长度进行校验
			return (value.length > 5)
		}
	}
}
```



props特性指定义的属性，在组件中有对应的接收，反之则是非props特性，非props特性的属性值会显示在组件的dom上。



在组件中，直接绑定原生事件（如click）会无效，会被视为是自定义事件，需要子组件绑定事件向外触发，如果想直接使用原生事件，可以添加`.native`，即 `@click.native="xxx"`



非父子组件间传递值，使用bus/总线

``` vue
<body>
    <div id="app">
        <test content="Apple"></test>
        <test content="Banana"></test>
    </div>
    <script>
        Vue.prototype.bus = new Vue()
        Vue.component("test", {
            props: ['content'],
            template: '<div @click="handleClick">{{selfContent}}</div>',
            data: function () {
                return {
                    selfContent: this.content
                }
            },
            methods: {
                handleClick: function () {
                    this.bus.$emit('change', this.selfContent)
                }
            },
            mounted: function () {
                let this_ = this;    // 该this作用域是当前组件实例
                this.bus.$on('change', function (mes) {  //此时的this是bus指向的Vue实例
                    this_.selfContent = mes;   
                })
            }
        })
        var vm = new Vue({
            el: "#app"
        })
    </script>
</body>
```

首先在Vue的原型上添加原型属性`bus`并且将它指向Vue的实例，在点击方法中，使用bus的vue实例中的$emit事件方法，触发事件并传递值，随后在mounted方法中，使用$on方法监听事件，事件触发后，执行函数。

（个人理解：因为emit和on都是在同一个实例中，所以值可以在该实例中传递，随后再将值赋到组件中）



插槽，在组件中想要插入dom元素必须使用插槽,否则元素会被忽略，插槽可以写入默认值，默认插槽（即组件中的dom元素没有slot属性）只有一个，可直接用<slot></solt>使用，具名插槽（dom元素中有slot="name"属性）可以有多个，在template中<slot name="name"></slot>使用，template中只可以有一个根元素，所以需要使用一个外层div包裹。



作用域插槽

``` vue
<body>
    <div id="root">
        <body-content>
            <template slot-scope="{scope}">
                <h1>{{scope.user.name}}</h1>
            </template>
        </body-content>
    </div>
    <script>
        Vue.component('body-content', {
            data: function () {
                return {
                    list: [1, 2, 4, 5],
                    user: {
                        name: 'Tom',
                        age: 18,
                        sex: 'Female'
                    }
                }
            },
            template: `<div>
                <slot :user=user></slot>
                </div>`
        })
        new Vue({
            el: "#root"
        })
    </script>
```

让插槽内容可以访问到组件中的数据，模板中的slot需要有包裹，在slot中传递的属性，将在template中的slot-scope中被接收。

在vue2.6中，slot-scope被废弃，使用`v-slot:default=“xxx”`取代，其中default指插槽名，如果只有默认插槽可以省略为v-slot（并且可以直接在组件名上使用）,但如果有多个（具名插槽）以上则必须用完整的语法（在template标签使用）。

``` vue
            <template v-slot:default="scope">
                <h1>{{scope.user.age}}</h1>
            </template>
```



动态组件，组件可以根据is提供的组件名进行动态转换。

``` vue
<component is="组件名"></component>
```



v-once,将内容存放在内存中，多次复用可提高效率

