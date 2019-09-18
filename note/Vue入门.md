# Vue入门

## 基础语法

创建vue实例 new Vue{}

``` vue
<body>
    <div id="root"> <h1>hello {{msg}}</h1></div>  //挂载点
    //挂载点内部的是模板
    <script>
        new Vue({
            el:"#root",    //el:"#root"接管哪一个页面元素
            //template:'<h1>hello {{msg}}</h1>',   模板也可以写在实例中
            data:{
                msg:"hello world"
            }
        })
    </script>
</body>
```

{{}}  插值表达式，也可以使用v-text和v-html插值。

``` vue
<h1> v-text = "number"></h1>     //会进行转义
<h1> v-html = "number"></h1>     //不会进行转义
```

绑定事件v-on：''（可以用@简写）,方法需要写在实例中的methods属性中,可以在方法中直接修改数据。

```vue
<h1> v-on:click = "number">msg</h1>   //@Click=''
new Vue({
	methods:{
	handleClick :function(){(this.msg='hahah')}
	}
})
```

属性绑定 v-bind:属性='绑定'，可以简写为":"

``` vue
<h1> v-bind:title = "title">msg</h1>  //"title"为data中数据
<h1> :title = "title">msg</h1> 
```

双向数据绑定 v-model="绑定"，单项绑定指类似插值表达式，只有data数据的改变会影响插值，但双向绑定时，另外一方的改变同样会影响data中的数据。

```vue
    <div id="root">
        <div> {{msg}}</div>
        <textarea v-model="msg"></textarea>  //value改变时，div中的msg也会改变
    </div>
```

计算属性，在实例中的computed属性添加，根据其它数据项计算出来的结果，当其它数据项改变时，才会重新计算，否则调用之前的缓存。

``` vue
            computed: {
                fullName:function(){
                    return this.firstName + ' ' + this.lastName
                }
            }
```

侦听器，使用watch属性添加,当fullName发生改变时，调用函数。

``` vue
            watch:{
                fullName:function(){
                    this.count++;
                }
            }
```

v-if,如果指为false时，则把dom删除，true则重新添加。

v-show,如果指为false时，display：none，true则删除该样式。

v-for,循环显示页面结构，“item”为变量名，“list”为数据的绑定。

``` vue
    <div id="root">
        <div v-if="show">hello world</div>
        <ul><li  v-for="(item,index) of list":key="index">{{item}}</li></ul>
        <button @click="handleClick">click</button>
    </div>
    <script>
        new Vue({
            el: "#root",
            data: {
                show: true,
                list:[1,2,3,4]
            },
            methods: {
                handleClick : function () {
                    this.show = !this.show;
                }
            }
        })
    </script>
```

## Vue中的组件

组件指页面上的某一部分

局部组件,需要在实例中添加components属性，然后在里面再注册局部组件。

``` vue
    <script>
        var TodoItem= {
            template:'<li>item<li>'
        }
        new Vue({
            el: "#root",
            components:{
                'todo-item':TodoItem
            }
        })
    </script>
```

全局组件,添加内容需要在组件上添加content，然后在组件定义中再加入props接收content。

``` vue
<body>
    <div id="root">
        <input v-model="inputValue">
        <button @click="handleClick">click</button>
        <todo-item v-for="(item,index) of list" 
        :key="index"
        :content="item"
        >
    </todo-item>
    </div>
    <script>
        Vue.component('todo-item', {
            props:['content'],
            template: '<li>{{content}}<li>'
        })
        new Vue({
            el: "#root",
            // components:{
            //     // 'todo-item':TodoItem
            // },
            data: {
                inputValue: '',
                list: []
            },
            methods: {
                handleClick: function () {
                    this.list.push(this.inputValue);
                    this.inputValue = '';
                }
            }
        })
    </script>
</body>
```

在Vue中，每个组件都是Vue的实例，被外层实例包括在内，外层实例中没有template时，就会搜索挂载点，将挂载点中的dom当作时template。

Vue中，父子组件是通过属性传值的

``` vue
<body>
    <div id="root">
        <input v-model="inputValue">
        <button @click="handleClick">click</button>
        <todo-item v-for="(item,index) of list" 
        :key="index"
        :content="item"
        :index="index"
        @delete="handleDelete"
        >
    </todo-item>
    </div>
    <script>
        Vue.component('todo-item', {
            props:['content','index'],
            template: '<li @click="handleClick">{{content}}<li>',
            methods:{
                handleClick : function(){
                    this.$emit('delete',this.index);
                }
            }
        })
        new Vue({
            el: "#root",
            data: {
                inputValue: '',
                list: []
            },
            methods: {
                handleClick: function () {
                    this.list.push(this.inputValue);
                    this.inputValue = '';
                },
                handleDelete: function (index) {
                    this.list.splice(index,1);
                }
            }
        })
    </script>
</body>
```

