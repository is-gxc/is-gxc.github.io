---
title: Vue2 核心基础
date: 2025-06-01 21:33:03
categories: Vue
tags: [前端, Vue2]
---

# 简单的Vue示例

1. 想让Vue工作，就必须创建一个Vue实例，且要传入一个配置对象
2. root容器里的代码依然符合html规范，只不过混入了一些特殊的Vue语法
3. root容器里的代码被称为【Vue模板】
4. Vue实例和容器是一一对应的
5. 真实开发中只有一个Vue实例，并且会配合着组件一起使用
6. {{xxx}}中的xxx要写js表达式，且xxx可以自动读取到data中的所有属性
7. 一旦data中的数据发生改变，那么模板中用到该数据的地方也会自动更新



注意区分：js表达式 和 js代码(语句)

1. 表达式：一个表达式会产生一个值，可以放在任何一个需要值的地方：

   (1). a

   (2). a+b

   (3). demo(1)

   (4). x == y ? 'a' : 'b'

2. js代码(语句)

   (1). if(){}

   (2). for(){}

``` html
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport"
          content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>第一次使用vue</title>
    <!--vue js生产版本production版本-->
    <script src="../js/vue.js"></script>
</head>
<body>
    <div id="root">
     <!--插值语法-->
     <!--vue模版-->
        <h1>hello, {{ name }}</h1>
    </div>
    <div class="test">
        <!--注意双花括号中可以写js表达式-->
        <!--
          js表达式：一个表达式生成一个值，放在任何一个需要值的地方
                例如 (1).a (2) 1+b (3) demo(1) （4) x===y ? 1 : 0

          js语句： if while for...
        -->
        <h1>{{name.toUpperCase()}},{{address}} {{1+1}} {{Date.now()}}</h1>
    </div>
    <script type="text/javascript">
       //hello小案例
       Vue.config.productionTip = false; //默认不提示
       //创建vue实例对象
       /**
        * 注意:一个vue实例不可能去接管多个容器，如果有多个容器的情况，vue事例永远只接管第一个容器
        * 不能多个vue实例同时来接管同一个容器，如果有多个的情况下永远是第一个vue实例来接管该容器
        * vue实例与容器直接的对应关系是一对一的
        */
       new Vue({
           //配置对象
           el: '#root',// document.getElementById('root') //element el指定当前vue实例为哪一个容器服务，值通常为css选择器格式
           data: {
               //data用来存储数据，以后会用函数来表示
               name:'zhangsan',
               age:21
           }
       });

       new Vue({
           el: '.test',
           data:{
               name: 'lisi',
               address: 'beijing'
           }
       })
    </script>
</body>
</html>
```



# 模板语法

**Vue模板语法有2大类：**

**1. 插值语法：**

- **功能**：用于解析标签体内容。
- **写法**：`{{xxx}}`，xxx是js表达式，且可以直接读取到data中的所有属性。

**2. 指令语法：**

- **功能**：用于解析标签（包括：标签属性、标签体内容、绑定事件等）。
- **举例**：
  - `v-bind:href="xxx"`
  - 简写形式：`:href="xxx"`（xxx同样要写js表达式，且可直接读取data中的所有属性）
- **备注**：Vue中有很多指令，形式均为 `v-????`，此处仅以 `v-bind` 为例说明。

``` html
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport"
          content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Vue模版语法的学习</title>
    <script src="../js/vue.js"></script>
</head>
<body>
   <div id="root">
       <!--
          插值语法一般用在标签体的值{{}}
          指令语法:用于解析标签(标签体,标签属性, 绑定事件)上，类似于v-bind
       -->
       <h1>插值语法</h1>
       <h3>你好, {{ name }} </h3>
       <hr/>
       <h1>指令语法</h1>
       <!--v-bind绑定指令，就把下面的url当成一个js表达式去执行了-->
       <a v-bind:href="url.toUpperCase()" v-bind:x="x">百度一下</a>
       <!-- v-bind: 还可以简写为: -->
       <a :href="url_beijing" :x="x">google 北京</a>
       <p>学校: <a :href="school.url.toString()">{{ school.name }}</a> </p>
   </div>
   <script type="text/javascript">
       new Vue({
           el: "#root",
           data: {
               name : 'jack',
               url : 'https://www.baidu.com',
               x: 'test v-bind',
               url_beijing: 'https://www.baidu.com',
               school: {
                   name: '家里蹲大学',
                   url:'https://www.zhangweishihundan.com/'
               }
           }
       })
   </script>
</body>
</html>

```



# 数据绑定

**Vue中有2种数据绑定的方式：**

1. **单向绑定(v-bind)：**数据只能从data流向页面。

2. **双向绑定(v-model)：**数据不仅能从data流向页面，还可以从页面流向data。



**备注：**

1. 双向绑定一般都应用在表单类元素上（如：input、select等）
2. 2.v-model:value 可以简写为 v-model，因为v-model默认收集的就是value值。



``` html
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport"
          content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>vue 数据绑定</title>
    <script src="../js/vue.js"></script>
</head>
<body>
    <div id="root">
        <!--
			Vue中有2种数据绑定的方式：
					1.单向绑定(v-bind)：数据只能从data流向页面。
					2.双向绑定(v-model)：数据不仅能从data流向页面，还可以从页面流向data。
						备注：
							1.双向绑定一般都应用在表单类元素上（如：input、select等）
							2.v-model:value 可以简写为 v-model，因为v-model默认收集的就是value值。
		 -->
        <label>
            单项数据绑定:
            <!--<input type='text' v-bind:value="name"/>-->
            <!--简写-->
            <input type='text' :value="name"/>
        </label>
        <label>
            双向数据绑定:
            <!--<input type='text' v-model:value="name"/>-->
            <input type='text' v-model="name"/>
        </label>
        <br/>
         <!--
         不是什么都可用v-model的 这里v-model不支持h1
         v-model只能应用在表单元素上(输入元素)，与用户交互(都有共同的value属性)
         -->
        <h1 v-bind:x="name">
            你好啊
        </h1>
    </div>
    <script type="text/javascript">
        //v-bind可以完成数据绑定(单项绑定)
        //v-model双向数据绑定
        //单项数据绑定 双向数据绑定
        Vue.config.productionTip = false;
        new Vue({
            el: '#root',
            data:{
                name: 'beijing'
            }
        })
    </script>
</body>
</html>
```



# Vue配置项中的 el 和 data 的两种写法

## el

1. new Vue时候配置el属性。
2. 先创建Vue实例，随后再通过vm.$mount('#root')指定el的值。

## data

1. 对象式

2. 函数式

   

**由Vue管理的函数(例如data)，一定不要写箭头函数，一旦写了箭头函数，this就不再是Vue实例了。**

```html
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport"
          content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>el和data的两种写法</title>
    <script src="../js/vue.js"></script>
</head>
<body>
   <div id="root">
       <!--
			data与el的2种写法
					1.el有2种写法
									(1).new Vue时候配置el属性。
									(2).先创建Vue实例，随后再通过vm.$mount('#root')指定el的值。
					2.data有2种写法
									(1).对象式
									(2).函数式
					3.一个重要的原则：
									由Vue管理的函数(例如data)，一定不要写箭头函数，一旦写了箭头函数，this就不再是Vue实例了。
		-->

       <h1>你好，{{ name }}</h1>
   </div>
   <script type="text/javascript">
       Vue.config.productionTip = false;
       //el的两种写法
       // const v = new Vue({
       //     // el: '#root', //第一种写法
       //     data: {
       //         name: '上海'
       //     }
       // });
       //
       // // console.log(v);
       // //关联root容器，用mount方法
       // setTimeout(() => {
       //     v.$mount('#root'); //第二种写法 挂载到页面上
       // }, 3000)

       //data的两种写法
       new Vue({
          el: '#root',
           //data的第一种写法:对象式
           // data: {
           //     name: 'shanghai'
           // }
           //第二种写法: 函数式(返回对象中包含你想渲染的模版数据)
           //学习组件的时候要是用函数式 这个函数是Vue来调用的
           // data: () => {
           //    console.log(`@@@@`, this); //此时this是window，因为箭头函数没有自己的this，它向外查找找到的window
           //    return ({
           //         name: 'shanghai'
           //     })
           // }
           //尽量不要写成剪头函数，因为会丢失this
           data (){
              console.log('@@@', this); //此时this是Vue
              return {
                  name: 'shanghai'
              }
           }
       });
   </script>
</body>
</html>
```



# MVVM

[MVVM模型](https://zh.wikipedia.org/wiki/MVVM)

**M：模型(Model)** ：data中的数据

**V：视图(View)** ：模板代码

**VM：视图模型(ViewModel)**：Vue实例



1.data中所有的属性，最后都出现在了vm身上。

2.vm身上所有的属性 及 Vue原型上所有属性，在Vue模板中都可以直接使用。

```html
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport"
          content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>mvvm模型的理解</title>
    <script src="../js/vue.js"></script>
</head>
<body>
    <div id="root">
        <!--view-->
        <h1>学校名称: {{ name }}</h1>
        <h1>学校地址:{{ address }}</h1>
        <h1>测试一下1:  {{ 1 + 1 }}</h1>
        <!--所有viewModel和vue原型上的属性在视图中的{{}}都可以看到-->
<!--        <h1>测试一下2: {{ $options }}</h1>-->
<!--        <h1>测试一下3: {{ $emit }}</h1>-->
<!--        <h1>测试一下4: {{ _c }}</h1>-->
    </div>
    <script type="text/javascript">
        //view model view与model之间的纽带
        //vm view实例对象
        const vm =  new Vue({
            el: '#root',
            data(){
                //model
                return {
                    name:'家里蹲大学',
                    address: '北京'
                }
            }
        });
        console.log(vm);
    </script>
</body>
</html>
```



# 数据代理

**Vue中的数据代理：**通过vm对象来代理data对象中属性的操作（读/写）

**Vue中数据代理的好处：**更加方便的操作data中的数据

**基本原理：**通过Object.defineProperty()把data对象中所有属性添加到vm上。为每一个添加到vm上的属性，都指定一个getter/setter。在getter/setter内部去操作（读/写）data中对应的属性。

``` html
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport"
          content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>vue中的数据代理</title>
    <script src="../js/vue.js"></script>
</head>
<body>
     <div id="root">
         <h2>学校名称:{{ name }}</h2>
         <h2>学校地址: {{ address }}</h2>
     </div>
    <script type="text/javascript">
        Vue.config.productionTip = false;
        let data;
        //通过vm代理
        const vm = new Vue({
           el:'#root',
            //vm._data === options.data = data
            //
           data(){
               return data = {
                   name:'zhangsan',
                   address:'beijing'
               }
           }
        });
    </script>
</body>
</html>
```

# 事件处理

## 事件的基本使用

1. **使用v-on:**xxx 或 @xxx 绑定事件，其中xxx是事件名
2. 事件的回调需要配置在**methods**对象中，最终会在vm上
3. **methods中配置的函数，不要用箭头函数！否则this就不是vm了**
4. methods中配置的函数，都是被Vue所管理的函数，**this的指向是vm 或 组件实例对象**
5. `@click="demo" `和 `@click="demo($event)"` 效果一致，但后者可以传参

``` html
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport"
          content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>事件处理基本</title>
    <script src="../js/vue.js"></script>
</head>
<body>
    <div id="root">
        <!--指令语法 v开头 例如v-on:click点击事件-->
        <h1>欢迎 {{ name }} </h1>
        <button v-on:click="showInfo1">点我提示信息1</button>
        <!--简写形式 @click-->
        <button @click="showInfo2($event,66)">点我提示信息2</button>
    </div>
    <script type="text/javascript">
        Vue.config.productionTip = false;
        const vm = new Vue({
            el: '#root',
            data(){
                return {
                   name: 'shanghai',

                }
            },
            //methods配置事件处理的回调函数
            methods:{
                showInfo1(e){
                    //默认给你传递event参数
                    //当是箭头函数的话this那就是window
                    console.log(this === vm); //this是vue实例
                    console.log('你好');
                },
                //接收参数
                showInfo2(e,num){
                    console.log(e.target);//
                    console.log(`接收参数：${num}`);
                }
            }
        });


    </script>
</body>
</html>
```

## Vue中的事件修饰符

1. **prevent**：阻止默认事件（常用）
2. **stop**：阻止事件冒泡（常用）
3. **once**：事件只触发一次（常用）
4. capture：使用事件的捕获模式
5. self：只有event.target是当前操作的元素时才触发事件
6. passive：事件的默认行为立即执行，无需等待事件回调执行完毕

``` html
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport"
          content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>修饰符</title>
    <script src="../js/vue.js"></script>
    <style>
        *{
            margin: 20px;
        }
        .demo1{
            height: 100px;
            background: deepskyblue;
        }
        .box1{
            padding: 5px;
            background: deepskyblue;
        }
        .box2 {
            padding: 5px;
            background: orange;
        }
        .list{
            width:200px;
            height: 200px;
            background: salmon;
            overflow: auto;
        }
        .list li{
            height: 100px;
        }
    </style>
</head>
<body>
   <div id="root">
       <h1>欢迎来到 {{ name }}</h1>
       <!--阻止默认事件（常用-->
       <a href="https://www.baidu.com" @click.prevent="showInfo" >点我提示信息</a>
       <!--阻止事件冒泡（常用）-->
       <div class="demo1" @click="showInfo">
           <button @click.stop="showInfo">点我提示信息</button>
           <!--修饰符也可以连用，例如下面事例是即阻止冒泡同时也阻止默认行为-->
           <!--<a href="https://www.google.com.tw" @click.prevent.stop="showInfo">谷歌台湾</a>-->
       </div>
       <!--事件只触发一次-->
       <button @click.once="showInfo">点我提示信息,只在第一次点击生效</button>
       <!-- capture事件的捕获模式 让事件以捕获的方式来处理(先捕获再冒泡)-->
       <div class="box1" @click.capture="showMsg(1)">
           div1
           <div class="box2" @click="showMsg(2)">div2</div>
       </div>
       <!-- self 只有event.target是当前操作的元素时才触发事件(变相阻止冒泡)-->
       <div class="demo1" @click.self="showInfo">
           <button @click="showInfo">点我提示信息</button>
       </div>
       <!--passive：事件的默认行为立即执行，无需等待事件回调执行完毕；-->
       <!--scroll滚动条一滚动就会触发的事件 wheel鼠标滚轮事件-->
       <ul class="list" @scroll.passive="demo">
           <li>1</li>
           <li>2</li>
           <li>3</li>
           <li>4</li>
       </ul>
   </div>
   <script type='text/javascript'>
       Vue.config.productionTip = false;
       new Vue({
           el: "#root",
           data(){
               return {
                   name: 'beijing'
               }
           },
           methods:{
               showInfo(e){
                   // e.preventDefault(); 阻止a标签默认行为
                   // e.stopPropagation() //阻止事件冒泡
                   // alert('你好');
                   console.log(e.target);
               },
               showMsg(msg){
                   console.log(msg);
               },
               demo(){
                   // console.log(`@`)
                   // for(let i = 0; i < 100000; i++){
                   //     console.log('#')
                   // }
                   // console.log('累了')
               }
           }
       });
   </script>
</body>
</html>
```

## Vue中的键盘事件

### Vue中常用的按键别名：

- 回车 => enter
- 删除 => delete (捕获“删除”和“退格”键)
- 退出 => esc
- 空格 => space
- 换行 => tab (特殊，必须配合keydown去使用)
- 上 => up
- 下 => down
- 左 => left
- 右 => right



Vue未提供别名的按键，可以使用按键原始的key值去绑定，但注意要**转为kebab-case（短横线命名）**

也可以使用keyCode去指定具体的按键（不推荐）

Vue.config.keyCodes.自定义键名 = 键码，可以去定制按键别名



### **系统修饰键（用法特殊）：**ctrl、alt、shift、meta

1. 配合keyup使用：按下修饰键的同时，再按下其他键，随后释放其他键，事件才被触发。
2. 配合keydown使用：正常触发事件。



<hr/>

```html
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport"
          content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>vue 键盘事件</title>
    <script src="../js/vue.js"></script>
</head>
<body>
  <div id="root">
        <!--vue中键盘事件的绑定 一般用keyUp(keydown)-->
        <h1>欢迎来到{{ name }}</h1>
        <input type="text" @keyup.enter="showInfo" placeholder="按下回车提示输入"/>
        <input type="text" @keyup.delete="showInfo" placeholder="退格或删除提示输入"/>
        <input type="text" @keyup.esc="showInfo" placeholder="按下esc提示输入"/>
        <input type="text" @keydown.tab="showInfo" placeholder="按下tab示输入"/>
        <input type="text" @keyup.ctrl="showInfo" placeholder="按下command提示输入"/>
        <input type="text" @keydown.p="showInfo" placeholder="按下p提示输入"/>
        <!--键盘事件的修饰符也可以连用-->
       <input type="text" @keydown.command.g="showInfo" placeholder="按下command和g提示输入"/>
    </div>
    <script type="text/javascript">
        new Vue({
            el:'#root',
            data:{
                name: 'shanghai'
            },
            methods:{
                showInfo:function (e){
                    // if(e.keyCode === 13) {
                    //     console.log(e.target.value);
                    // }
                    console.log(e.target.value);
                    console.log(e.key); //按下的名字
                }
            }
        })
    </script>
</body>
</html>
```



# 计算属性

定义：要用的属性不存在，要通过已有属性计算得来。

原理：底层借助了Objcet.defineproperty方法提供的getter和setter。

get函数什么时候执行？

1. 初次读取时会执行一次。

2. 当依赖的数据发生改变时会被再次调用。

   

优势：与methods实现相比，内部有缓存机制（复用），效率更高，调试方便。

备注：

1. 计算属性最终会出现在vm上，直接读取使用即可。

2. 如果计算属性要被修改，那必须写set函数去响应修改，且set中要引起计算时依赖的数据发生改变。



``` html
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport"
          content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>vue计算属性语法实现</title>
    <script src="../js/vue.js"></script>
</head>
<body>
<div id="root">
    <!--v-model双向绑定-->
    姓:<input type="text" v-model="firstName"/>
    <br/><br/>
    名:<input type="text" v-model="lastName"/>
    <br/><br/>
    测试:<input type="text" v-model="test"/>
    <br/><br/>
    全名: <span>{{ fullName }}</span>
</div>
<script type="text/javascript">
    const vm = new Vue({
        el: '#root',
        data: {
            //data里的都是属性
            firstName: '张',
            lastName: '三',
            test:'test'
        },
        //计算属性--> 旧属性加工
        computed: {
            fullName: {
                //get的作用，当读取fullName时get就会被调用，且返回值就做为fullName的值
                //defineProperty
                //注意vm._data是不存在计算属性的
                //计算属性算完之后直接丢到了viewModel实例对象身上
                /**
                 * get的调用时机
                 * 1.初次读取计算属性时
                 * 2.所依赖的数据(data中的属性)发生变化时，不改变的话直接读缓存
                 * 3.methods没有缓存，读几次就调用几次无论要用的数据是否发生变化
                 *  @returns {string}
                 */
                get(){
                    //此时get函数中的this指向是vm
                    console.log('get调用了', this);
                    return this.firstName + '--' + this.lastName
                },
                /**
                 * set什么时候调用
                 *   1.当fullName被修改时
                 * @param value
                 */
                set(value){
                    //修改计算属性所依赖的普通属性(放在data里面的)
                    //this === vm
                    const [ firstName, lastName ] = value.split('-');
                    this.firstName = firstName;
                    this.lastName = lastName; //依赖属性发生改变之后,计算属性自动改变
                }
            }
        }
    });
</script>
</body>
</html>
```



**计算属性简写语法实现**

``` html
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport"
          content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>vue计算属性语法实现</title>
    <script src="../js/vue.js"></script>
</head>
<body>
<div id="root">
    <!--v-model双向绑定-->
    姓:<input type="text" v-model="firstName"/>
    <br/><br/>
    名:<input type="text" v-model="lastName"/>
    <br/><br/>
    测试:<input type="text" v-model="test"/>
    <br/><br/>
    全名: <span>{{ fullName }}</span>
</div>
<script type="text/javascript">
    const vm = new Vue({
        el: '#root',
        data: {
            //data里的都是属性
            firstName: '张',
            lastName: '三',
            test:'test'
        },
        //计算属性--> 旧属性加工
        computed: {
            //简写形式
            //前提:计算属性只考虑读取不考虑修改 set丢了
            //简写计算书写为一个函数(这个函数当成getter使用), 注意不要写箭头函数
            //执行完getter之后，vm直接保存返回的数据为fullName属性的属性值,此时vm.fullName不是函数而是一个确切的计算值
            fullName: function (){
                return this.firstName + '--' + this.lastName;
            }
     }});
</script>
</body>
</html>
```



# 监视属性

 当被监视的属性变化时, 回调函数自动调用, 进行相关操作

监视的属性必须存在，才能进行监视！！

监视的两种写法：

1. new Vue时传入watch配置
2. 通过vm.$watch监视

``` html
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport"
          content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>天气案例_监视属性</title>
    <script src="../js/vue.js"></script>
</head>
<body>
<div id="root">
    <h1>今天天气很 {{ info }}</h1>
    <button @click="changeWeather">
        切换
    </button>
</div>
<script type="text/javascript">
    Vue.config.productionTip = false;
    const vm = new Vue({
        el:'#root',
        data:{
            isHot: true
        },
        //计算属性
        computed: {
            info(){
                return this.isHot ? '炎热' : '凉爽';
            }
        },
        //改变模版数据的方法
        methods:{
            changeWeather(){
                this.isHot = !this.isHot;
            }
        },
        // watch:{
        //     //监视的配置对象
        //     //watch不仅能监视data的普通属性，也可以检测计算属性
        //     isHot:{
        //         immediate: true, //当这个属性为true时，页面刚渲染就运行handler函数
        //         //handler 什么时候调用呢
        //         //当isHot发生改变就会调用该函数
        //         //handler接收两个参数，一个是这个状态参数改变前的值，另一个是改变后的旧值
        //         handler(newValue, preValue){
        //             console.log('ishot 被修改了');
        //             console.log(`newValue: ${newValue}, preValue: ${preValue}`);
        //         }
        //     }
        // }
    });
    //watch的第二种写法，直接采用vm对象实例
    vm.$watch('isHot', {
        immediate: true, //当这个属性为true时，页面刚渲染就运行handler函数
        //handler 什么时候调用呢
        //当isHot发生改变就会调用该函数
        //handler接收两个参数，一个是这个状态参数改变前的值，另一个是改变后的旧值
        handler(newValue, preValue){
            console.log('ishot 被修改了');
            console.log(`newValue: ${newValue}, preValue: ${preValue}`);
        }
    });
</script>
</body>
</html>
```



## 深度监视

深度监视：

1. Vue中的watch默认不监测对象内部值的改变（一层）。

2. 配置deep:true可以监测对象内部值改变（多层）。

   

备注：

1. Vue自身可以监测对象内部值的改变，但Vue提供的watch默认不可以(想让它可以则配置deep:true)！
2. .使用watch时根据数据的具体结构，决定是否采用深度监视。

``` html
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport"
          content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>天气案例_深度监视</title>
    <script src="../js/vue.js"></script>
</head>
<body>
<div id="root">
    <h1>今天天气很 {{ info }}</h1>
    <button @click="changeWeather">
        切换
    </button>
    <hr/>
    <h3>a的值是:{{ numbers.a }}</h3>
    <button @click="numbers.a++">
        点我让a加一
    </button>
    <h3>b的值是:{{ numbers.b }}</h3>
    <button @click="numbers.b++">
        点我让b加一
    </button>
    <h1>
        测试vue自身监测数据属性
        {{ numbers.d.e }}
    </h1>
</div>
<script type="text/javascript">
    Vue.config.productionTip = false;
    const vm = new Vue({
        el:'#root',
        data:{
            isHot: true,
            numbers:{
                a:1,
                b:1,
                d:{
                    e:2
                }
            }
        },
        //计算属性
        computed: {
            info(){
                return this.isHot ? '炎热' : '凉爽';
            }
        },
        //改变模版数据的方法
        methods:{
            changeWeather(){
                this.isHot = !this.isHot;
            }
        },
        watch:{
            //监视的配置对象
            //watch不仅能监视data的普通属性，也可以检测计算属性
            isHot:{
                //immediate: true, //当这个属性为true时，页面刚渲染就运行handler函数
                //handler 什么时候调用呢
                //当isHot发生改变就会调用该函数
                //handler接收两个参数，一个是这个状态参数改变前的值，另一个是改变后的旧值
                handler(newValue, preValue){
                    console.log('ishot 被修改了');
                    console.log(`newValue: ${newValue}, preValue: ${preValue}`);
                }
            },
            //监测多级结构里面的属性 深度监视
            // 'numbers.a':{
            //     handler(){
            //       console.log('a发生改变了')
            //     }
            // }
            //深度监视numbers中的所有属性
            numbers:{
                deep: true, //监视多级结构的属性
                handler(){
                    console.log('numbers 发生改变')
                }
            }
        }
    });

</script>
</body>
</html>
```

## 监视属性的简写形式

``` html
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport"
          content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>天气案例_深度监视简写形式</title>
    <script src="../js/vue.js"></script>
</head>
<body>
<div id="root">
    <h1>今天天气很 {{ info }}</h1>
    <button @click="changeWeather">
        切换
    </button>
    <hr/>
</div>
<script type="text/javascript">
    Vue.config.productionTip = false;
    const vm = new Vue({
        el:'#root',
        data:{
            isHot: true,
        },
        //计算属性
        computed: {
            info(){
                return this.isHot ? '炎热' : '凉爽';
            }
        },
        //改变模版数据的方法
        methods:{
            changeWeather(){
                this.isHot = !this.isHot;
            }
        },
        // watch:{
        //     //监视的配置对象
        //     //watch不仅能监视data的普通属性，也可以检测计算属性
        //     // isHot:{
        //     //     //immediate: true, //当这个属性为true时，页面刚渲染就运行handler函数
        //     //     //handler 什么时候调用呢
        //     //     //当isHot发生改变就会调用该函数
        //     //     //handler接收两个参数，一个是这个状态参数改变前的值，另一个是改变后的旧值
        //     //     //deep: true  //简写
        //     //     handler(newValue, preValue){
        //     //         console.log('ishot 被修改了');
        //     //         console.log(`newValue: ${newValue}, preValue: ${preValue}`);
        //     //     }
        //     // }
        //     //简写:
        //     //简写的前提watch的属性不需要immediate和deep属性的时候
        //     // isHot(newValue, preValue){
        //     //     console.log(newValue,preValue);
        //     // }
        // }
    });
    //完整写法
    // vm.$watch('isHot', {
    //     deep: true,
    //     immediate: true,
    //     handler(newValue, preValue){
    //         console.log(newValue, preValue);
    //     }
    // });
    //简写 (代价就是不能配置其他配置项deep immediate)
    vm.$watch('isHot', function (newValue, preValue){
        //this === vm
        console.log(newValue, preValue);
    })
</script>
</body>
</html>
```

## 计算属性和监视属性之间的区别

computed能完成的功能，watch都可以完成。

watch能完成的功能，computed不一定能完成，例如：watch可以进行异步操作。



两个重要的小原则：

1. **所被Vue管理的函数，最好写成普通函数，这样this的指向才是vm 或 组件实例对象。**
2. **所有不被Vue所管理的函数（定时器的回调函数、ajax的回调函数等、Promise的回调函数），最好写成箭头函数，这样this的指向才是vm 或 组件实例对象。**



# 绑定样式

## class样式

写法:class="xxx" xxx可以是字符串、对象、数组。

1. 字符串写法适用于：类名不确定，要动态获取。

2. 对象写法适用于：要绑定多个样式，个数不确定，名字也不确定。

3. 数组写法适用于：要绑定多个样式，个数确定，名字也确定，但不确定用不用。

## style样式
`:style="{fontSize: xxx}"`其中xxx是动态值。
`:style="[a,b]"`其中a、b是样式对象。



<hr/>

``` html
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport"
          content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>vue绑定样式</title>
    <style>
        .basic{
            width: 400px;
            height: 100px;
            border: 1px solid black;
        }

        .happy{
            border: 4px solid red;;
            background-color: rgba(255, 255, 0, 0.644);
            background: linear-gradient(30deg,yellow,pink,orange,yellow);
        }
        .sad{
            border: 4px dashed rgb(2, 197, 2);
            background-color: gray;
        }
        .normal{
            background-color: skyblue;
        }

        .atguigu1{
            background-color: yellowgreen;
        }
        .atguigu2{
            font-size: 30px;
            text-shadow:2px 2px 10px red;
        }
        .atguigu3{
            border-radius: 20px;
        }

    </style>
    <script src="../js/vue.js"></script>
</head>
<body>
<div id="root">
    <!--:class 绑定class样式字符串写法 适用于样式的类名不确定需要动态琢磨的状况-->
    <div class="basic" :class="mood"  @click="changeMood">{{ name }}</div>
    <hr/>
    <!--:class 绑定class样式数组写法 适用于要绑定的样式个数不确定，名字也不确定的状况-->
    <div class="basic" :class="classArr">{{ name }}</div>
    <hr/>
    <!--:class 绑定class样式对象写法 适用于要绑定的样式个数确定，名字确定，但动态决定要不要用的状况-->
    <div class="basic" :class="classObj" >{{ name }}</div>
    <hr/>
    <!--绑定style样式 对象写法-->
    <div class="basic" :style="styleObj">
        {{ name }}
    </div>
    <hr/>
    <!--绑定style样式 数组写法-->
    <div class="basic" :style="[styleObj, styleObj2]">
        {{ name }}
    </div>
    <hr/>
    <div class="basic" :style="styleArr">
        {{ name }}
    </div>
</div>
<script type="text/javascript">
    new Vue({
       el:'#root',
       data: {
           name:'test',
           mood:null,
           classArr:['atguigu1', 'atguigu2', 'atguigu3'],
           classObj:{
               atguigu1: false,
               atguigu2: false
           },
           styleObj:{
               fontSize: '40px',
               color: 'red',
           },
           styleObj2:{
               background: 'green'
           },
           styleArr:[
               { color: 'orange' },
               { background: 'grey' }
           ]
       },
       methods:{
         changeMood(){
             //vue绑定事件
             //不要亲手操作dom
             //随机切换心情
             const moodsArr = ['normal', 'happy', 'sad'];
             const random = Math.floor(Math.random() * moodsArr.length);
             this.mood = moodsArr[random];
         }
       }
    });
</script>
</body>
</html>
```



# 条件渲染

## v-if

写法：

1. v-if="表达式"
2. v-else-if="表达式"
3. v-else="表达式"



适用于：切换频率较低的场景。

特点：不展示的DOM元素直接被移除。

注意：v-if可以和:v-else-if、v-else一起使用，但要求结构不能被“打断”。



## v-show

写法：v-show="表达式"

适用于：切换频率较高的场景。

特点：**不展示的DOM元素未被移除，仅仅是使用样式隐藏掉**

<hr/>

**备注：使用v-if的时，元素可能无法获取到，而使用v-show一定可以获取到。**



```html
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport"
          content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>条件渲染</title>
    <script src="../js/vue.js"></script>
</head>
<body>
<div id="root">
    <!--v-show要能转换为一个布尔值 v-show条件渲染-->
    <!--    <h2 v-show="a">欢迎来到{{ name }}</h2>-->

    <!--    <h2 v-show="1===1">欢迎来到{{ name }}</h2>-->
    <!--    <button @click='a = !a'>{{ a ? '隐藏': '显示' }}</button>-->
        <!--使用v-if来进行条件渲染 -->
    <!--    <h2 v-if="1">欢迎来到{{ name }}</h2>-->
        <!--v-if v-else-if v-else记住不能打断，要连着写-->
    <!--    <h2>当前的n值是{{ n }}</h2>-->
        <button @click="n++">点我n+1</button>
    <!--    <div v-if="n === 1">angular</div>-->
    <!--    <div v-else-if="n === 2">react</div>-->
    <!--&lt;!&ndash;    <div>111</div>&ndash;&gt;-->
    <!--    <div v-else-if="n === 3">vue js</div>-->
    <!--    <div v-else>not found</div>-->

    <!--v-if与template的配合使用-->
    <template v-if="n === 1">
        <h2>你好</h2>
        <h2>shanghai</h2>
        <h2>shenzhen</h2>
    </template>
</div>
<script type="text/javascript">
    const vm = new Vue({
        el: '#root',
        data:{
            name: 'shanghai',
            n:0
        }
    })
</script>
</body>
</html>
```



# 列表渲染

## v-for指令

1. 用于展示列表数据
2. 语法：`v-for="(item, index) in xxx" :key="yyy"`
3. 可遍历：数组、对象、字符串（用的很少）、指定次数（用的很少）

``` html
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport"
          content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>基本列表</title>
    <script src="../js/vue.js"></script>
</head>
<body>
<div id="root">
    <h2>人员列表</h2>
    <ul>
        <!--遍历数组-->
        <!--循环列表的方法 类似与for in循环遍历-->
        <!--:代表v-bind 属性key让每一个li有了唯一的标识,key一定不要重复-->
        <!--v-for(for in// for of)可以接受到两个参数,一个是当前的元素另一个是当前元素的索引 类似于下面的person,index-->
        <li v-for='(person, index) in persons' :key="index">
            <!--p可能来自形参，也可能来自于写在data里的属性，更可能来自于计算属性 computed-->
            {{person.name}} - {{ person.age }}
        </li>
    </ul>
    <!--遍历对象-->
    <h2>汽车信息</h2>
    <!--注意遍历对象的时候先收到的是每一项的属性的value，第二项是对应的键名:key-->
    <ul v-for="(val, key) of car" :key="key">
         <li>{{ key }} -- {{ val }}</li>
    </ul>
    <!--遍历字符串 用的不多-->
    <h2>测试遍历字符串</h2>
    <!--注意遍历字符串的时候先收到的是字符串中每一个字符，第二项是其对应的索引index-->
    <ul v-for="(c, index) of str" :key="index">
        <li>{{ index }} -- {{ c }}</li>
    </ul>
    <!--遍历指定次数-->
    <h2>遍历指定次数</h2>
    <!--注意遍历指定次数的时候先收到的是number(例如5，则number是1，2，3，4，5)，第二项是对应index(从0开始计数,则是0,1,2,3,4)-->
    <ul v-for="(num, index) in 5" :key="index">
        <li>{{ index }} -- {{ num }}</li>
    </ul>
</div>
<script type="text/javascript">
    Vue.config.productionTip = false;
    new Vue({
        el: '#root',
        data: {
            //数组
            persons: [
                {id: '001', name: '张三', age: 18},
                {id: '002', name: '李四', age: 19},
                {id: '003', name: '王五', age: 20}
            ],
            car: {
                name: '奥迪a8',
                price: '70w',
                color: '黑色'
            },
            str: 'hello'
        }
    })
</script>
</body>
</html>
```



## key 有什么作用？

虚拟DOM中key的作用:

key是虚拟DOM对象的标识，当状态中的数据发生变化时，Vue会根据【新数据】生成【新的虚拟DOM】,随后Vue进行【新虚拟DOM】与【旧虚拟DOM】的差异比较，比较规则如下:
对比规则：

1. 旧虚拟DOM中找到了与新虚拟DOM相同的key:
   a. 若虚拟DOM中内容没变,直接使用之前的真实DOM !

   b. 若虚拟DOM中内容变了，则生成新的真实DOM，随后替换掉页面中之前的真实DOM.

2. 旧虚拟DOM中未找到与新虚拟DOM相同的key：创建新的真实:DOM，随后渲染到到页面。



用index作为key可能会引发的问题：若对数据进行:逆序添加、逆序删除等破坏顺序操作:会产生没有必要的真实DOM更新==界面效果没问题,但效率低。如果结构中还包含输入类的DOM:会产生错误DOM更新==>界面有问题。

开发中如何选择key?：最好使用每条数据的唯一标识作为key,比如id、手机号、身份证号、学号等唯一值。如果不存在对数据的逆序添加、逆序删除等破坏顺序操作，仅用于渲染列表用于展示，使用index作为key是没有问题的。



# 过滤器

定义：对要显示的数据进行特定格式化后再显示（适用于一些简单逻辑的处理）。

语法：

1. 注册过滤器：`Vue.filter(name,callback)` 或 `new Vue{filters:{}}`

2. 使用过滤器：`{{ xxx | 过滤器名}}`  或 ` v-bind:属性 = "xxx | 过滤器名"`



备注：

1. 过滤器也可以接收额外参数、多个过滤器也可以串联
2. 并没有改变原本的数据, 是产生新的对应的数据

``` html 
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport"
          content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>vue中过滤器</title>
    <script src="../js/vue.js"></script>
    <script src="https://cdn.bootcdn.net/ajax/libs/dayjs/1.10.6/dayjs.min.js"></script>
</head>
<body>
<div id="root">
    <h1>显示格式化后的时间</h1>
    <!--计算属性实现-->
    <h2>现在是:{{ fmtTime }}</h2>
    <!--methods实现-->
    <h2>现在是{{ getFmtTime() }}</h2>
    <!--过滤器实现-->
    <h2>现在是:{{ time |  timeFormater }}</h2>
    <!--过滤器也可以传递参数-->
    <h2>现在是:{{ time | timeFormater('YYYY-MM-DD') | mySlice }}</h2>
    <h3 :x="msg | mySlice">你好,世界</h3>
</div>
<div id="root2">
   <h2>{{ msg | mySlice }}</h2>
</div>
<script type="text/javascript">
    Vue.config.productionTip = false;
    //全局过滤器的配置
    //注意配置一定要new vue实例之前确定
    Vue.filter('mySlice', function (val){
        return val.slice(0,4);
    });
    new Vue({
        el: "#root",
        data:{
            time: 1631808423062,
            msg: "你好，世界"
        },
        computed:{
            fmtTime(){
                return dayjs(this.time).format('YYYY-MM-DD HH:mm:ss')
            }
        },
        methods:{
            getFmtTime(){
                return dayjs(this.time).format('YYYY-MM-DD HH:mm:ss')
            }
        },
        //局部过滤器
        filters:{
            //过滤器本质上也是一个函数
            timeFormater(val, formate = 'YYYY-MM-DD HH:mm:ss'){
                return dayjs(val).format(formate)
            },
        }
    });

    const vm2 = new Vue({
        el:"#root2",
        data:{
            msg:'welcome'
        }
    })
</script>
</body>
</html>
```



# 内置指令

## v-text

作用：向其所在的节点中渲染文本内容。 (纯文本渲染)

与插值语法的区别：v-text会替换掉节点中的内容，{{xx}}则不会。这里有点不太灵活



## v-html

作用：向指定节点中渲染包含html结构的内容。

与插值语法的区别：

1. v-html会替换掉节点中所有的内容，{{xx}}则不会。
2. v-html可以识别html结构。



严重注意：v-html有安全性问题！！！！

1. 在网站上动态渲染任意HTML是非常危险的，容易导致XSS攻击。
2. 一定要在可信的内容上使用v-html，永不要用在用户提交的内容上！



## v-cloak

v-cloak指令（没有值）：

1. 本质是一个特殊属性，Vue实例创建完毕并接管容器后，会删掉v-cloak属性。
2. 使用css配合v-cloak可以解决网速慢时页面展示出{{xxx}}的问题。



## v-once

v-once所在节点在初次动态渲染后，就视为静态内容了。

以后数据的改变不会引起v-once所在结构的更新，可以用于优化性能。



## v-pre

跳过其所在节点的编译过程。

可利用它跳过：没有使用指令语法、没有使用插值语法的节点，会加快编译。



# 自定义指令

定义语法：

1. 局部指令：new Vue({directives:{指令名:配置对象} })   或  new Vue({directives: {指令名:回调函数}})
2. 全局指令：Vue.directive(指令名,配置对象) 或  Vue.directive(指令名,回调函数)



备注：

1. 指令定义时不加v-，但使用时要加v-
2. 指令名如果是多个单词，要使用kebab-case命名方式，不要用camelCase命名。
3. 写的指令函数中，第一个参数代表真是的dom元素，第二个参数是当定对象，有例如value的属性。在绑定函数中，this是window，vue 实例对象。

``` html
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport"
          content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>自定义指令</title>
    <script src="../js/vue.js"></script>
</head>
<body>
<!--
   需求1：定义一个v-big指令，和v-text功能类似，但会把绑定的数值放大10倍。
   需求2：定义一个v-fbind指令，和v-bind功能类似，但可以让其所绑定的input元素默认获取焦点。
   自定义指令总结：
			一、定义语法：
						(1).局部指令：
							new Vue({directives:{指令名:配置对象} })   或  new Vue({directives: {指令名:回调函数}})
						(2).全局指令：
							Vue.directive(指令名,配置对象) 或  Vue.directive(指令名,回调函数)

			二、配置对象中常用的3个回调：
						(1).bind：指令与元素成功绑定时调用。
						(2).inserted：指令所在元素被插入页面时调用。
						(3).update：指令所在模板结构被重新解析时调用。

			三、备注：
				         1.指令定义时不加v-，但使用时要加v-；
						 2.指令名如果是多个单词，要使用kebab-case命名方式，不要用camelCase命名。
-->
<!--
  下面的这是模版，要经过vue的解析才能放到页面中，dom树里
-->
<div id="root">
    <h2>当前的n值是：<span v-text="n"></span></h2>
<!--    <h2>放大10倍后的n值是: <span v-big-number="n"></span></h2>-->
    <h2>放大10倍后的n值是: <span v-big="n"></span></h2>
    <button @click="n++">点我n+1</button>
    <p>测试指令函数所调用的时机: {{ name }} </p>
    <hr/>
    <input type="text" v-fbind:value="n"/>
</div>
<div id="root2">
    <input type="text" v-fbind:value="x"/>
</div>
<script type="text/javascript">
    Vue.config.productionTip = false;
    //此时自定义fbind指令使全局指令了，其他vue实例所管理的容器也可以使用
    //全局指令
    Vue.directive('fbind', {
        bind(el, binding){
            // console.log('bind')
            el.value = binding.value;
        },
        //指令被插入页面时
        inserted(el, binding){
            // console.log('inserted')
            el.focus();
        },
        //指令所在模版被重新解析时
        update(el, binding){
            // console.log('update');
            el.value = binding.value;
        }
    })
    const vm = new Vue({
        el:"#root",
        data:{
            name: '上海',
            n:1
        },
        //定义指令的配置项: directives
        directives:{
            /**
             big函数的调用时机:
                    1.指令与元素成功绑定(但注意此时dom元素还没有成功的被弄到页面上去)时会被调用 (首次,类似于dom元素一加载)
                    2.指令所在的模版被重新解析时会被再次调用
            **/
            //两种写法:1.对象(key-value) 2.函数
            // 'big-number'(el,binding){
            //     console.log('big被调用啦！')
            //     //收到两个参数，第一个参数代表真实dom元素，第二个参数是绑定对象，关注该绑定对象中的value属性
            //     // console.log(el,binding);
            //     //原生dom的操作
            //     el.innerText = binding.value * 10;
            // },
            big(el,binding){
                console.log(this); //注意此处this===window vue实例对象此时'不管'你写在指令里面的函数了
                console.log('big被调用啦！')
                //收到两个参数，第一个参数代表真实dom元素，第二个参数是绑定对象，关注该绑定对象中的value属性
                // console.log(el,binding);
                //原生dom的操作
                el.innerText = binding.value * 10;
            },
            //自定义fbind绑定
            //换成对象写法 对比函数简写方式其实只是多了 inserted钩子
            // fbind:{
            //     //指令与元素成功绑定
            //     // bind(el, binding){
            //     //     // console.log('bind')
            //     //     el.value = binding.value;
            //     // },
            //     // //指令被插入页面时
            //     // inserted(el, binding){
            //     //     // console.log('inserted')
            //     //     el.focus();
            //     // },
            //     // //指令所在模版被重新解析时
            //     // update(el, binding){
            //     //     // console.log('update');
            //     //     el.value = binding.value;
            //     // }
            // }
        }
    });

    const vm2 = new Vue({
        el:'#root2',
        data:{
            x: 1,
        }
    })
</script>
</body>
</html>
```



# 生命周期

Vue组件的生命周期是理解其运行机制的核心。每个组件从创建到销毁会经历一系列阶段，Vue在这些阶段提供了**生命周期钩子函数**，方便开发者插入自定义逻辑。

## 核心阶段与钩子函数

1. **创建阶段**
   - `beforeCreate`：实例刚创建，**数据观测未初始化**（无法访问data/methods）。
   - `created`：实例创建完成，**数据已注入**（可操作data、调用方法），**但DOM未生成**（避免操作DOM）。
2. **挂载阶段**
   - `beforeMount`：编译模板完成，**虚拟DOM已生成，真实DOM未替换**。
   - `mounted`：组件已挂载到页面，**真实DOM渲染完成**（可安全操作DOM或发起异步请求）。
3. **更新阶段**（数据变化时触发）
   - `beforeUpdate`：数据更新后，**虚拟DOM重新渲染前**（可获取更新前状态）。
   - `updated`：虚拟DOM重新渲染并应用至真实DOM后（**避免在此修改数据，可能引发循环更新**）。
4. **销毁阶段**
   - `beforeDestroy`：实例销毁前触发（**清理定时器、解绑事件**，防止内存泄漏）。
   - `destroyed`：实例完全销毁，所有绑定和监听被移除。

## 关键实践建议

- **异步请求**：在 `created` 或 `mounted` 中发起（需考虑SSR则选created）。
- **DOM操作**：必须在 `mounted` 后进行。
- **性能优化**：避免在 `updated` 中修改响应式数据。
- **资源清理**：在 `beforeDestroy` 中移除全局事件监听和定时器。

常用的生命周期钩子：

1. mounted: 发送ajax请求、启动定时器、绑定自定义事件、订阅消息等【初始化操作】。
2. beforeDestroy: 清除定时器、解绑自定义事件、取消订阅消息等【收尾工作】。
