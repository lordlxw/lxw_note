# Vue

##  一、简介

VUE：一套用于构建用户界面的渐进式JS框架

React——Angular——Vue

Vue中一个.vue文件就是一个组件（html+css+js）



加一层：实现复用（Diff算法）

![image-20220801155405104](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20220801155405104.png)



两种方式使用Vue：

:one:使用Script标签（或是CDN）

:two:CLI NPM包



补充：Ctrl+F5或Shift+F5 可以强制刷新界面



### 1、Vue实例配置

> el：element，表示元素，常用el:'#root' //代表一个名称为root的容器  el用于指定当前Vue实例为哪个容器服务。值通常为css id选择器。
>
> el的两种写法：el是固定绑定了，也可以先创建vue实例再用mount挂载到某一元素上
>
> data的两种写法：对象式 和 函数式 data:function(){return {name:'lxw'}} 函数式必须要有返回值，组件必须要用函数式
>
> data：{} 键值对形式，用于存储数据
>
> ## Vue中容器和实例是一一对应关系

```Vue
 <div id="root">
        <h1>Hello{{name}},{{address}},年龄:{{a+b}},{{Date.now()}}</h1>
 </div>
    <script type="text/javascript">
        Vue.config.productionTip=false;

        //创建Vue实例
        const x = new Vue({
            el:'#root',   //css id选择器 el用于指定当前Vue实例为哪个容器服务。值通常为css id选择器
            data:{
                name:'小牛',
                address:'上海虹口',
                age:'20',
                a:1100,
                b:2000
                
            }
        });
```



### 2、插值语法 和 指令语法

模板语法分为：插值语法 和 指令语法

> 指令语法：用v-bind:href 绑定href  <a v-bind:href="url">点我去百度</a>
>
> vue中指令就是 v-开头
>
> #### *v-bind: ----->简写为 ：
>
> 插值语法往往指定标签体内容，
>
> 指令语法往往用于解析标签（标签属性、标签体内容、绑定事件）



### 3、MVVM

两个指令可以完成数据绑定

v-bind:    单向数据绑定

v-model: 双向数据绑定 （只能用于表单类元素（输入）中）



Vue就是view-model 





### 4、数据代理

> Object.defineproperty(类名，要增加的属性名，value值)
>
> 可以对一个类增加属性，控制属性能否枚举——enumerable:true (默认为false)  
>
> ​										  控制属性是否重写——writable:true(默认为false)
>
> get——有人读取时调用
>
> set——有人设置时调用

---

数据代理：通过一个对象代理对另一个对象中属性的操作（读/写）

```Vue
<script>
     let obj={x:100};
     let obj2={y:200};
     Object.defineProperty(obj2,'x',{
            get(){
                return obj.x;
            },
            set(value){
                console.log("在设置值中...")
                obj.x=value;
            }
        })
</script>
```



> vm._data.name  
>
> get、set访问器
>
> 响应式，个人感觉就跟wpf的数据绑定一样，监测到数据发生变化，然后数据劫持，修改界面上的值



### 5、事件处理

> <button v-on:click>点击</button>
>
> v-on:click=“showInfo”           或    	 	@click（简写）
>
> 点击调用showinfo方法 ， 注意此方法也要放在vue实例methods里

### 6、事件修饰符

> 1. prevent：阻止默认事件（常用）
> 2. stop：阻止事件冒泡（常用）
> 3. once：事件只触发一次（常用）
> 4. capture：使用事件的捕获模式
> 5. self：只有event.target是当前操作的元素时才触发事件
> 6. passive：事件的默认行为立即执行，无需等待事件回调执行，可以用在滚轮上。（不等事件回调完成就执行）



### 7、Vue常用按键别名

> 回车：enter
>
> 删除：delete
>
> 退出：esc
>
> 空格：space
>
> 换行：tab
>
> 上：up
>
> 下：down
>
> 左：left
>
> 右：right

> keydown和keyup
>
> 13代表enter，不建议用编码表示（不同键盘编码一般不相同）
>
> ctrl、win等多个键一起按需要用keyup 		ctrl.y——表示ctrl+y键一起按时才触发



### 8、计算属性

> 要用的属性不存在，通过已有的属性计算获得
>
> 关键字：computed
>
> 底层：借助了 object.defineproperty
>
> 优势：get只在初次读取时执行，当依赖数据发生变化时会再次调用get
>
> 与methods实现相比，内部有缓存机制，效率高，调试方便

> ### 注意：计算属性有缓存

```Vue
const vm = new Vue({
	el:'root',
	data:{
		firstName:'zhang',
		lastName:'san'
		}
	computed:{
		fullName:{
		get(){
			return this.firstName+'-'=this.lastName 
			}
		set(value)
			{
			const arr=value.split('-')
			this.firstName=arr[0]
			this.lastName=arr[1]
			}	
		}

	}
})
```

通常计算属性用到get，set不常用



```Vue
computed:{
	fullName(){
		return this.firstName+'-'=this.lastName 
	}
}
```



### 9、监视属性

```vue
CONST VM = new Vue({
		el:'#root'
		watch:{
			监视名:{
				handler(){
						console.log("变化时触发")
					}
				}
			}

})

```



handler——当监视名属性（计算属性也可以）发生变化时，触发。



watch里的监视名方法里加 deep:true 开启深度监视模块，可以监视多层级内部的数据

> vm.$watch('监视属性名',{方法}) //这样比较灵活

watch能开启异步任务，计算属性不行



监视属性和计算属性的区别，计算属性就是一旦属性变就触发（要靠返回值），而监视属性可以控制延迟一秒等操作



### 10、条件渲染

> v-show = “true” ,显示 ，false为隐藏，其实内部就是css中display:none
>
> v-if ——比较狠，整个结构干掉，比较耗性能
>
> v-else-if——正常逻辑一样，但可以省接下去的判断
>
> v-else——同上
>
> 注意：之前的语句要连续，不能被打断
>
> template模板（就是划分模块）只能跟v-if配合使用







### 11、Vue中key的作用和原理

> key不能重复，能提高性能
>
> v-for遍历
> 先出现value，再出现key

> 原理：diff算法（对比）
>
> ![image-20220803124416054](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20220803124416054.png)





注意：特殊情况下不能以index为key，最好主键（唯一标识）为key。因为若对数据打乱顺序，会出问题

默认情况下不指定key就以index为标识（类似数组下标）





### 12、列表过滤

> 一个字符串默认包含空串
>
> 优先使用计算属性实现





###  13、Vue监测数据变化的原理

> vue如何监测对象数据改变？
>
> vue首先加工data，将每个属性加上get、set方法，这样才能实现响应式，底层是深度递归，会将子元素的各个属性也都添加get、set方法（深拷贝）
>
> 界面name属性修改实际后台操作：
>
> name被修改-->调用setter-->重新解析模板-->生成新的虚拟DOM-->新旧DOM对比（diff）-->更新页面

> vue如何监测数组数据改变？
>
> :one: 在数据忘记添加子项时，Vue.set可以日后追加属性，但只能给data里的对象添加属性，不能给data的根属性添加
>
> vue.set(vm._data.student,'sex','男')
>
> 或者vm.$set(vm._data.student,'sex','男') ——两者实现一致
>
> :two:数组这个东西内部元素vue不会自动分配get、set方法，而是只对整个数组分配一个get、set，就是说不能通过数组索引值修改数组内部值。
>
> #### 所以说数组的修改是重写
>
> Vue将侦听的数组变更方法进行了包装，一共七个，对数组中某个元素进行操作一定要用以下七个方法之一：
>
> push、pop、shift、unshift、splice、sort、reverse
