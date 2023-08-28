# 1.vue 2 简介

- JavaScript框架
- 简化Dom操作
- 响应式数据驱动

官方文档：[安装 — Vue.js (vuejs.org)](https://v2.cn.vuejs.org/v2/guide/installation.html)



# 2.第一个Vue程序

- 导入开发版本的Vuejs

  - ```html
    <script src="https://cdn.jsdelivr.net/npm/vue@2/dist/vue.js"></script>
    ```

- 创建Vue实例对象,设置el属性和data属性

  - ```html
    <div id="app">
        {{ message }}
    </div>
    <script>
        var app = new Vue({
            el: "#app",
            data: {
                message: "你好， 小兔"
            }
        })
    </script>
    ```

  - 

- 使用简洁的模板语法把数据渲染到页面上



# 3.el挂载点

- Vue会管理el选项命中的元素及其内部的后代元素
- 可以使用其他的选择器,但是建议使用ID选择器
- 可以使用其他的双标签不能使用HTML和BODY

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Vue基础</title>
</head>
<script src="https://cdn.jsdelivr.net/npm/vue@2/dist/vue.js"></script>
<body>
    {{ message }}
    <div id="app" class="app">
        {{ message }}
        <span>{{ message }}</span>
    </div>
    <script>
        var app = new Vue({
            // el: "#app",
            // el:".app",
            el:"div",
            data: {
                message: "你好， 小兔"
            }
        })
    </script>
</body>
</html>
```



# 4.data数据对象

- Vue中用到的数据定义在data中
- data中可以写复杂类型的数据
- 渲染复杂类型数据时,遵守js的语法即可

```html
<body>
    <div id="app">
        {{ message }}
        {{ school }}
        <h2>{{school.name}}</h2>
        <h2>{{school.dog}}</h2>
        <ul>
            <li>{{sex[0]}}</li>
            <li>{{sex[1]}}</li>
        </ul>
    </div>
    <script>
        var app = new Vue({
            el:"#app",
            data: {
                message: "hello vue",
                sex: ["男","女"],
                school: {
                    name: "husky",
                    dog: "二哈"
                }
            }
        })
    </script>
</body>
```



# 5.本地应用

- 通过Vue实现常见的网页效果
- 学习Vue指令,以案例巩固知识点
- Vue指令指的是以v-开头的一组特殊语法



## 5.1 v-text指令

- v-text指令的作用是:设置标签的内容(textContent)
- 默认写法会替换标签内部的所有内容,使用差值表达式{{}}可以换指定内容
- 内部支持写表达式

```html
<script src="https://cdn.jsdelivr.net/npm/vue@2/dist/vue.js"></script>
<body>
    <div id="app">
        <h2 v-text="message + '!'">soft</h2>
        <h2 v-text="info+ '!'">soft</h2>
        <h2>{{info + "!"}}soft</h2>
    </div>
</body>
<script>
    var app = new Vue({
        el:"#app",
        data:{
            message:"v-text指令",
            info:"东软"
        }
    })
</script>
```

<img src="D:\Java学习\java笔记\vue\vue.assets\image-20221213155109668.png" alt="image-20221213155109668" style="zoom:67%;" />



## 5.2 v-html

- v-html指令的作用是:设置元素的innerHTML
- 内容中有html结构会被解析为标签
- v-text指令无论内容是什么只会解析为文本
- 解析文本使用v-text,需要解析html结构使用v-html

```html
<body>
    <div id="app">
        <h2 v-text="content"></h2>
        <h2 v-html="content">百度</h2>
    </div>
</body>
<script>
    var app = new Vue({
        el:"#app",
        data:{
            content:"<a href='http://www.baidu.com'>百度</a>"
        }
    })
</script>
```

<img src="D:\Java学习\java笔记\vue\vue.assets\image-20221213161946005.png" alt="image-20221213161946005" style="zoom:50%;" />



## 5.3 v-on指令

- v-on指令的作用是:为元素绑定事件
- 事件名不需要写on
- 指令可以简写为@
- 绑定的方法定义在methods属性中

```html
<body>
    <div id="app">
        <!-- 单击事件 -->
        <input type="button" value="v-on指令" v-on:click="doIt"/>
        <input type="button" value="v-on简写" @click="doIt"/>
        <!-- 双击事件 -->
        <input type="button" value="双击事件" @dblclick="doIt"/>
        <h2 @click="changeRabbit">{{message}}</h2>
    </div>
</body>
<script>
    var app = new Vue({
        el:"#app",
        data:{
            message: "兔子"
        },
        methods:{
            doIt:function(){
                alert("事件")
            },
            changeRabbit:function(){
                this.message+="好好吃"
            }
        }
    })
</script>
```

### 5.3.1 案例

- 创建Vue示例时时:el(挂载点),data(数据),methods(方法)
- v-on指令的作用是绑定事件，简写为@
- 方法中通过this关键字获取data中的数据
- v-text指令的作用是:设置元素的文本值简写为{{}}
- v-html指令的作用是:设置元素的innerHTML



**计数器例子：**

```html
<body>
    <div id="app" class="input-num">
        <button @click="removeNum">-</button>
        <span>{{message}}</span>
        <button @click="addNum">+</button>
    </div>
</body>
<script>
    var app = new Vue({
        el:".input-num",
        data:{
            message:0
        },
        methods: {
            removeNum:function(){
                if(this.message > 0) {
                    this.message--;                     
                }else{
                    alert("别点啦，最小啦")
                }
            },
            addNum:function(){
                if(this.message < 10) {
                    this.message++;                     
                }else{
                    alert("别点啦，最大啦")
                }
            }
        }
    })
</script>
```



## 5.4 v-show指令

- v-show指令的作用是:根据真假切换元素的显示状态
- 原理是修改元素的display,实现显示隐藏
- 指令后面的内容,最终都会解析为布尔值
- 值为true元素显示,值为false元素隐藏
- 数据改变之后,对应元素的显示状态会同步更新

```html
<body>
    <div id="app">
        <input type="button" value="切换状态" @click="changeIsShow"><br>
        <img src="./image/red.jpg" v-show="isShow" width="300"><br>
        <input type="button" value="累加年龄" @click="addAge"><br>
        <img src="./image/p2871449782 (3).jpg" v-show="isAge>=18" width="300">
    </div>
</body>
<script>
    var app = new Vue({
        el:"#app",
        data:{
            isShow: true,
            isAge: 17
        },
        methods:{
            changeIsShow:function(){
                this.isShow = !this.isShow
            },
            addAge:function(){
                this.isAge++
            }
        }
    })
</script>
```



## 5.5 v-if指令

- v-if指令的作用是:根据表达式的真假切换元素的显示状态
- 本质是通过操纵dom元素来切换显示状态
- 表达式的值为true,元素存在于dom树中,为false,从dom树中移除
- 频繁的切换使用v-show,反之使用v-if;前者的切换消耗小

```html
<body>
    <div id="app">
        <input type="button" value="切换状态" @click="changeIsShow"><br>
        <p v-if="isShow">husky</p>
        <p v-show="isShow">husky v-show</p>
        <h2 v-if="temperature>=35">{{message}}</h2>
    </div>
</body>
<script>
    var app = new Vue({
        el:"#app",
        data:{
            isShow: true,
            message:"热死啦",
            temperature:35
        },
        methods:{
            changeIsShow:function(){
                this.isShow = !this.isShow
            }
        }
    })
</script>
```



## 5.6 v-bind指令

- v-bind指令的作用是:为元素绑定属性
- 完整写法是 v-bind:属性名
- 简写的话可以直接省略v-bind,只保留 :属性名
- 需要动态的增删class建议使用对象的方式

```html
<style>
    .active{
        border: 2px solid red;
    }
</style>
<body>
   <div id="app">
        <img v-bind:src="imageSrc" alt="" width="200"><br>
        <img :src="imageSrc" alt="" width="200" :title="imageTitle">
        <img :src="imageSrc" alt="" width="200" :title="imageTitle" 
        :class="{active:isActive}" @click="activeCli">
   </div> 
</body>
<script>
    var app = new Vue({
        el:"#app",
        data:{
            imageSrc:"./image/red.jpg",
            imageTitle:"红发",
            isActive:false
        },
        methods:{
            activeCli:function(){
                this.isActive = !this.isActive;
            }
        }
    })
</script>
```

### 5.6.1案例

- 列表数据使用数组保存
- v-bind指令可以设置元素属性,比如src
- v-show和v-if都可以切换元素的显示状态频繁切换用v-show
- 切换图片案例：

```html
<body>
    <div id="app">
        <img :src="imgSrc[index]" alt="" width="200" :class=""><br>
        <input type="button" value="上一张" v-show="index!=0" @click="prev"/>
        <input type="button" value="下一张" v-show="index!=imgSrc.length-1" @click="next"/>
    </div>
</body>
<script>
    var app = new Vue({
        el:"#app",
        data:{
            imgSrc:["./image/微信图片_20221114205645.png",
                    "./image/one.jpg",
                    "./image/p2871449782 (1).jpg",
                    "./image/p2871449782 (2).jpg",
                    "./image/preview.jpg",
                    "./image/red.jpg"],
            index:0
        },
        methods:{
            prev:function(){
                this.index--;
            },
            next:function(){
                this.index++;
            }
        }
    })
</script>
```



## 5.7 v-for指令

- v-for指令的作用是:根据数据生成列表结构
- 数组经常和v-for结合使用
- 语法是( item,index) in 数据
- item和index可以结合其他指令一起使用
- 数组长度的更新会同步到页面上,是响应式的

```html
<template>
  <div>
    <ul>
      <li v-for="(item, index) in list" :key="index">{{item}}</li>
    </ul>
    <button @click="changeList">改变数组</button>
  </div>
</template>

<script>
export default {
  name: "数组",
  data() {
    return {
      list: [1,2,3,4,5,6]
    }
  },
  methods:{
    changeList:function () {
      // 在数组末尾添加元素
      this.list.push(7, 8);

      // 删除数组最后一个元素
      this.list.pop();

      // 删除数组第一位
      this.list.shift();

      // 给数组首位添加元素
      this.list.unshift(0);

      // splice();//删除元系、插入元素、替换元素
      // 第一个参数:表示开始插入或者开始删除的元素的位置下标
      // 删除元素
      // 第二个参数:表示传入要删除几个元素(如果没有传，就删除后面所有的元素)
      // 插入元素:
      // 第二个参数:传入0,并且后面接上要插入的元素
      // 替换元素:
      // 第二个参数:表示我们替换几个元素，后面的参数表示用于替换前面的元素的
      this.list.splice(1)

      // 排序
      this.list.sort();

      // 翻转
      this.list.reverse();
    }
  }
}
</script>

<style>

</style>

```



## 5.8 v-on补充

- 事件绑定的方法写成函数调用的形式，可以传入自定义参数
- 定义方法时需要定义形参来接收传入的实参
- 事件的后面跟上.修饰符可以对事件进行限制
- enter 可以限制触发的按键为回车
- 事件修饰符有多种

```html
<script src="https://cdn.jsdelivr.net/npm/vue@2/dist/vue.js"></script>
<body>
    <div id="app">
        <input type="button" value="v-on" @click="doIt('皮卡丘',123)"></input>
        <input type="text" @keyup.enter="sayHi"></input>
    </div>
</body>
<script>
    var app = new Vue({
        el:"#app",
        methods:{
            doIt:function(p,a){
                alert("干掉他"+p+a);
            },
            sayHi:function(){
                alert("aaaaaaaaa");
            }
        }
    })
</script>
```



## 5.9 v-model指令

- v-model指令的作用是便捷的设置和获取表单元素的值
- 绑定的数据会和表单元素值相关联
- 绑定的数据和表单元素的值是双向绑定的

```java
<template>
<!-- 双向绑定 -->
  <div>
    <h2>{{msg}}</h2>
    <input type="text" v-model="msg">
  </div>
<!-- 单个勾选框 -->
  <input type="checkbox" v-model="checked">
  <h2>{{checked}}</h2>
<!-- 多个勾选框 -->
  <input type="checkbox" v-model="fruits" value="苹果">苹果
  <input type="checkbox" v-model="fruits" value="雪梨">雪梨
  <input type="checkbox" v-model="fruits" value="荔枝">荔枝
  <input type="checkbox" v-model="fruits" value="西瓜">西瓜
  <input type="checkbox" v-model="fruits" value="龙眼">龙眼
  <h2>选择的水果有：{{fruits}}</h2>
<!-- 单选框 -->
  <input type="radio" v-model="sex" value="男">男
  <input type="radio" v-model="sex" value="女">女
  <h2>{{sex}}</h2>
<!-- 选项框 -->
<!-- 单选 -->
  <select v-model="city">
    <option value="广州">广州</option>
    <option value="深圳">深圳</option>
    <option value="东莞">东莞</option>
    <option value="茂名">茂名</option>
  </select>
  <h2>{{city}}</h2>
<!-- 多选 -->
  <select v-model="cities" multiple>
    <option value="广州">广州</option>
    <option value="深圳">深圳</option>
    <option value="东莞">东莞</option>
    <option value="茂名">茂名</option>
  </select>
  <h2>{{cities}}</h2>
</template>

<script>
export default {
  name: "表单输入绑定",
  data() {
    return {
      msg: "helloWorld",
      checked: true,
      fruits: [],
      sex: "男",
      city: "",
      cities: []
    }
  },
}
</script>

<style>

</style>
```



### 5.9.1记事本案列

- 列表结构可以通过v-for指令结合数据生成
- v-on结合事件修饰符可以对事件进行限制,比如.enter
- v-on在绑定事件时可以传递自定义参数
- 通过v-model可以快速的设置和获取表单元素的值
- 基于数据的开发方式

```html
<body>
    <!-- 主体区域 -->
    <section id="todoapp">
        <header class="header">
            <h1>husky记事本</h1>
            <!-- 输入框 -->
            <input  placeholder="请输入任务" 
            v-model="message" @keyup.enter="addM" 
            autofocus="autofocus" autocomplete="off"/>
        </header>
        <!-- 列表区域 -->
        <section class="main">
            <ul>
                <li v-for="(item,index) in arrM" @click="remove(index)">{{index+1+"."}}{{item}}</li>
            </ul>
        </section>
        <!-- 统计和清除 -->
        <footer class="footer">
            <input v-show="arrM.length!=0" type="button" value="clear" @click="removeAll"/>
            <h6 v-show="arrM.length!=0">总共{{arrM.length}}数据</h6>
        </footer>
    </section>
</body>
<script>
    var app = new Vue({
        el:"#todoapp",
        data:{
            message:"",
            arrM:[]
        },
        methods:{
            addM:function(){
                this.arrM.push(this.message);
            },
            removeAll:function(){
                this.arrM.splice(0, this.arrM.length);
            },
            remove:function(i){
                // splice(下标,从下标开始删几个)
                this.arrM.splice(i, 1);
            }
        }
    })
</script>
```



# 6.网络应用

## 6.1 axios

- axios必须先导入才可以使用
- 使用get或post方法即可发送对应的请求
- then方法中的回调函数会在请求成功或失败时触发
- 通过回调函数的形参可以获取响应内容,或错误信息

```html
<script src="https://unpkg.com/axios/dist/axios.min.js"></script>
```

```javascript
axios.get(地址?key=value&key2=value2).then(function(response){},function(err){})
axios.post(地址,{key:value,key2:value2}).then(function(response){},function(err){})
```

