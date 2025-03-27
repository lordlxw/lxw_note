> 对于vite构建，index.html才是入口文件
>
> **严格来说，Vite 的入口文件是 `index.html`**，因为 Vite 是基于 **原生 ES 模块** 的开发服务器，它直接在 `index.html` 中加载 `src/main.js` 或 `src/main.ts`。





### OptionsAPI 和 CompositionAPI

> Vue2 是选项式的其实就是背选项，然后写
>
> ## **Vue 2 Options API 的主要弊端**
>
> ### **1️⃣ 代码逻辑分散，维护困难**
>
> 在 Options API 中，**同一功能的代码可能分散在多个选项中**，比如：
>
> - `data` 负责存储状态
> - `computed` 处理派生数据
> - `methods` 定义函数
> - `watch` 监听变化
>
> 当组件变复杂后，不同功能的代码会分散在各个选项里，导致代码阅读和维护变得困难。
>
> ### **2️⃣ 代码复用困难 (Mixins 问题)**
>
> Vue 2 使用 **Mixins** 复用代码，但它有很多弊端：
>
> - **数据来源不明确**，多个 mixins 可能会定义相同的变量，导致命名冲突
> - **难以追踪数据来源**，你不知道 `this.someData` 是从组件本身还是某个 mixin 来的
>
> 
>
> 
>
> 
>
> Vue3的API是**组合**方式的
>
> 在 **Vue 3** 的 **Composition API** 中，`this` 的指向和 **Options API** 中的 `this` 不同。因为在 **Composition API** 中，`setup()` 函数没有 `this`，所以无法直接通过 `this` 访问组件的实例或数据。
>
> ### **1. Composition API 中的 `this`**
>
> 在 `setup()` 函数内，**`this` 是 `undefined`**，因为 `setup()` 函数本身不依赖于组件实例。
>
> - 你不能直接使用 `this` 来访问组件中的数据、方法等。
> - `setup()` 函数的 **返回值** 变成了模板中可用的数据或方法，而不是通过 `this` 来访问。
>
> ### **示例：Composition API**
>
> ```
> vue复制编辑<script setup>
> import { ref } from "vue";
> 
> // 创建响应式数据
> const name = ref("张三");
> 
> // setup() 中没有 this
> console.log(this); // undefined
> </script>
> 
> <template>
>   <h1>{{ name }}</h1>
> </template>
> ```
>
> ### **2. Options API 中的 `this`**
>
> 在 **Options API** 中，`this` 是组件实例本身，可以通过 `this` 访问组件中的所有数据、方法、计算属性等。
>
> - `data`、`computed`、`methods` 等都可以通过 `this` 来访问。
>
> ### **示例：Options API**
>
> ```
> vue复制编辑<script>
> export default {
>   data() {
>     return {
>       name: "张三"
>     };
>   },
>   mounted() {
>     console.log(this.name); // 通过 this 访问数据
>   }
> };
> </script>
> 
> <template>
>   <h1>{{ name }}</h1>
> </template>
> ```





> composingApi中默认数据不是==响应式==的
>
> 在 **Vue 3** 中的 **Composition API** 中，数据是通过 `ref()` 或 `reactive()` 来定义的，这些数据同样是 **响应式** 的。





### setup()

> 在 Vue 3 中，`setup()` 是 Composition API 的核心部分，用于组织组件的逻辑和数据。`setup()` 返回的值在模板中具有特殊意义，这些值会暴露给组件的模板和其他 Composition API 函数。接下来，我将详细讲解 `setup()` 的返回值的作用和使用。
>
> ### **1. `setup()` 的基本结构**
>
> `setup()` 函数是 **Vue 3** 组件生命周期的一部分，作为组件实例的初始化函数，它在组件实例化之前调用。
>
> ```vue
> <script setup>
> import { ref } from 'vue';
> 
> const name = ref('张三');
> const age = ref(18);
> 
> // 返回值会暴露给模板
> return { name, age };
> </script>
> 
> <template>
>   <div>
>     <h1>{{ name }}</h1>
>     <h2>{{ age }}</h2>
>   </div>
> </template>
> ```
>
> ### **2. `setup()` 返回值的作用**
>
> 在 `setup()` 中 **返回的对象会暴露给模板**，这些返回的属性、方法、计算属性等可以直接在模板中使用。返回值在模板中等同于 Vue 组件的 `data`、`computed`、`methods` 等，但这些都是由 `setup()` 函数组织和暴露出来的。
>
> #### **2.1 返回的数据（`data`）**
>
> `setup()` 返回的数据就像 Vue 2 中的 `data`，可以是基本类型、对象、数组等，并且它们是响应式的。
>
> ```vue
> <script setup>
> import { ref } from 'vue';
> 
> const name = ref('张三');
> const age = ref(18);
> 
> return { name, age };
> </script>
> 
> <template>
>   <div>
>     <h1>{{ name }}</h1>
>     <h2>{{ age }}</h2>
>   </div>
> </template>
> ```
>
> - 在 `setup()` 中返回的 `name` 和 `age` 会自动成为响应式，模板会自动根据这些数据的变化重新渲染。
>
> #### **2.2 返回的方法（`methods`）**
>
> 你可以在 `setup()` 中定义方法并返回，这些方法也会暴露到模板中。
>
> ```vue
> <script setup>
> import { ref } from 'vue';
> 
> const name = ref('张三');
> const age = ref(18);
> 
> const changeName = () => {
>   name.value = '李四';
> };
> 
> return { name, age, changeName };
> </script>
> 
> <template>
>   <div>
>     <h1>{{ name }}</h1>
>     <h2>{{ age }}</h2>
>     <button @click="changeName">改变名字</button>
>   </div>
> </template>
> ```
>
> - 在模板中，可以直接使用返回的 `changeName` 方法。
>
> #### **2.3 计算属性（`computed`）**
>
> 你也可以使用 Vue 3 的 `computed()` 来创建计算属性，并在 `setup()` 中返回。
>
> ```vue
> <script setup>
> import { ref, computed } from 'vue';
> 
> const age = ref(18);
> 
> const doubleAge = computed(() => age.value * 2);
> 
> return { age, doubleAge };
> </script>
> 
> <template>
>   <div>
>     <h1>{{ age }}</h1>
>     <h2>{{ doubleAge }}</h2>
>   </div>
> </template>
> ```
>
> - `doubleAge` 是一个计算属性，自动依赖 `age` 的变化，并在 `age` 改变时更新。
>
> #### **2.4 引用对象和数组（`ref` 和 `reactive`）**
>
> 你可以返回一个 `ref` 或 `reactive` 的对象，这些数据会保持响应式。
>
> ```vue
> <script setup>
> import { reactive } from 'vue';
> 
> const user = reactive({
>   name: '张三',
>   age: 18
> });
> 
> return { user };
> </script>
> 
> <template>
>   <div>
>     <h1>{{ user.name }}</h1>
>     <h2>{{ user.age }}</h2>
>   </div>
> </template>
> ```
>
> - `user` 是一个通过 `reactive` 创建的响应式对象，可以直接在模板中访问其属性。
>
> ### **3. 返回值的作用域**
>
> 返回值不仅在模板中可用，而且可以在 `setup()` 以外的地方使用，如 Vue 3 提供的其他 Composition API 函数中。
>
> #### **3.1 返回的值在 `setup()` 中的作用域：**
>
> - `setup()` 返回的所有数据、方法、计算属性等都可以直接在模板中使用，并且它们是响应式的。
> - 返回值的作用域是整个组件，不需要通过 `this` 访问，直接通过返回的属性访问即可。
>
> #### **3.2 通过 `setup()` 暴露的属性可以与其他 API 结合使用：**
>
> ```vue
> <script setup>
> import { ref, onMounted } from 'vue';
> 
> const name = ref('张三');
> 
> onMounted(() => {
>   console.log(name.value); // 访问返回的数据
> });
> </script>
> ```
>
> - `name` 可以在 `onMounted` 生命周期钩子中访问，甚至可以与其他 Composition API 函数一起使用。
>
> ### **4. 不返回任何内容时：**
>
> 如果 `setup()` 没有显式返回任何内容，则默认值是 `undefined`，此时你就无法在模板中访问任何数据或方法。
>
> ```vue
> <script setup>
> import { ref } from 'vue';
> 
> const name = ref('张三');
> </script>
> 
> <template>
>   <div>
>     <h1>{{ name }}</h1> <!-- 无法访问，name 不会暴露 -->
>   </div>
> </template>
> ```
>
> ### **5. 总结**
>
> - **`setup()` 返回的对象**：返回的值（数据、方法、计算属性等）会暴露到模板中，直接在模板中使用。
> - **响应式数据**：通过 `ref()` 或 `reactive()` 创建的数据，都会是响应式的。
> - **作用域**：返回的值可以在 `setup()` 内部使用，也可以传递给其他 Composition API 函数或生命周期钩子。
> - **没有返回值时**：如果不返回任何值，模板中将无法访问到数据和方法。
>
> `setup()` 返回的值是 Vue 3 Composition API 中非常重要的一部分，理解它的工作方式对于开发 Vue 3 应用至关重要。





> 生命周期钩子：
>
> 1. beforeCreate（已移除，在Vue 3中使用setup）
> 2. created（已移除，在Vue 3中使用setup）
> 3. beforeMount
> 4. mounted
> 5. beforeUpdate
> 6. updated
> 7. beforeUnmount
> 8. unmounted
> 9. errorCaptured
>
> setup是最早的
>
> vue2的options语法可以读取setup里的data和methods，（即用this访问到setup里的）反之不行，



> ==语法糖==：
>
> <script setup>
>     
> </script>
>
> 通过defineOptions来暴露name
>
> 
>
> ```vue
> <script setup>
> defineOptions({
>   name: 'MyComponent'  // 设置组件的 name
> });
> </script>
> 
> <template>
>   <div>
>     <h1>组件名称：{{ name }}</h1>
>   </div>
> </template>
> 
> ```
>
> 





### 响应式数据 ref & reactive

> <script lang="ts" setup>
> import { ref } from "vue";
> let name = ref('张三');
> let age = ref(18);
> </script>





> ref是用于定义 ==基本类型==
>
> 
>
> 模板里无需加.value
>
> ```vue
>  <h1>姓名：{{ name }}</h1>
>     <h1>年龄：{{ age }}</h1>
> ```
>
> 但如果要修改数据，要加上.value





---

> reactive是用于定义==对象类型的==响应式数据
>
> import {reactive} from 'vue'
>
> ![image-20250301112650992](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20250301112650992.png)





> 对比：
>
> ### **主要区别**
>
> | 特性               | `ref`                                          | `reactive`                                 |
> | ------------------ | ---------------------------------------------- | ------------------------------------------ |
> | **适用场景**       | 基本数据类型（字符串、数字、布尔值）或简单对象 | 对象、数组及其嵌套结构                     |
> | **访问方式**       | 通过 `.value` 访问                             | 直接访问，无需 `.value`                    |
> | **深度响应**       | 只有值本身是响应式，不会深度代理对象内部的属性 | 深度代理对象或数组的所有嵌套属性           |
> | **组合与嵌套使用** | 对于复杂对象需要使用 `ref` 和 `.value`         | 对象和数组本身都是响应式，可以直接修改     |
> | **性能**           | 对于基本类型，`ref` 更加轻量和高效             | 适用于处理嵌套对象或数组，性能稍逊于 `ref` |
>
> 
>
> ref 更全能：其实都能调用，也可以定义对象
>
> ref 和 reactive 的响应式原理不同，reactive 利用 Proxy代理对象，ref 利用的还是vue2的原理
>
> reactive只能定义对象类型
>
> ![image-20250301114442934](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20250301114442934.png)

> vue official插件可以自动补全：
>
> 即遇到ref的自动 加上.value
>
> ![image-20250301115119286](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20250301115119286.png)
>
> 如果想改，请用Object.Assign（）分配对象
>
> 
>
> ![image-20250301120029312](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20250301120029312.png)





### toRef & toRefs

> 将对象结构成 响应式对象
>
> 在 Vue 3 中，`toRefs` 和 `toRef` 都是用于将响应式数据转换为引用（`ref`）的一些工具，它们常用于组合式 API（Composition API）中，特别是在与响应式对象一起使用时。
>
> ### 1. **`toRefs`**
>
> `toRefs` 是一个用于将响应式对象的每个属性转化为 `ref` 的函数。它允许你保持对象的响应性，同时又能将每个属性转化为 `ref`，方便解构时仍能保持响应式。
>
> #### 语法：
>
> ```js
> import { toRefs } from 'vue';
> 
> const obj = reactive({ name: 'Alice', age: 30 });
> const { name, age } = toRefs(obj);
> ```
>
> #### 解释：
>
> - `toRefs` 主要用于将一个 **响应式对象**（`reactive`）的属性转为 `ref`，确保解构时属性的响应性得以保持。
> - 例如，如果你直接解构一个响应式对象，你可能会丢失它的响应性，但通过 `toRefs`，你可以确保每个解构出来的属性仍然是响应式的。
>
> #### 示例：
>
> ```js
> import { reactive, toRefs } from 'vue';
> 
> export default {
>   setup() {
>     const user = reactive({ name: 'Alice', age: 30 });
>     const { name, age } = toRefs(user);
> 
>     return {
>       name,
>       age,
>     };
>   },
> };
> ```
>
> - 在这个例子中，`user` 是一个响应式对象。如果我们直接使用 `name` 和 `age`，它们会失去响应性。但使用 `toRefs` 后，`name` 和 `age` 都变成了 `ref`，并保持响应式。
>
> ### 2. **`toRef`**
>
> `toRef` 是用于将 **响应式对象** 的单个属性转为 `ref`。它的作用和 `toRefs` 类似，但是它只处理对象的一个属性。
>
> #### 语法：
>
> ```js
> import { toRef } from 'vue';
> 
> const obj = reactive({ name: 'Alice', age: 30 });
> const nameRef = toRef(obj, 'name');
> ```
>
> #### 解释：
>
> - `toRef` 将一个响应式对象的单一属性转为 `ref`，你可以在函数内部只选择其中一个属性转换成 `ref`，并保持它的响应性。
> - 它的主要用途是将响应式对象的某个特定属性转化为 `ref`，尤其是在需要将一个对象的特定属性传递给其他组件或函数时。
>
> #### 示例：
>
> ```js
> import { reactive, toRef } from 'vue';
> 
> export default {
>   setup() {
>     const user = reactive({ name: 'Alice', age: 30 });
>     const nameRef = toRef(user, 'name');
> 
>     return {
>       nameRef,
>     };
>   },
> };
> ```
>
> - 在这个例子中，`toRef` 将 `user` 对象中的 `name` 属性转换成了一个 `ref`，并返回它。`nameRef` 现在可以像普通的 `ref` 一样使用。
>
> ### 3. **区别**
>
> - **`toRefs`**：将响应式对象中的 **所有属性** 转换为 `ref`，并返回一个包含所有 `ref` 的对象。
> - **`toRef`**：将响应式对象中的 **单个属性** 转换为 `ref`，并返回该属性的 `ref`。
>
> #### 示例比较：
>
> ```js
> import { reactive, toRefs, toRef } from 'vue';
> 
> const user = reactive({ name: 'Alice', age: 30 });
> 
> // 使用 toRefs 将整个对象的属性转为 ref
> const { name, age } = toRefs(user);
> 
> // 使用 toRef 转换单个属性
> const nameRef = toRef(user, 'name');
> ```
>
> - `toRefs(user)` 会将 `user` 对象的 `name` 和 `age` 都转为 `ref`。
> - `toRef(user, 'name')` 只会将 `user` 对象的 `name` 转为 `ref`。
>
> ### 总结：
>
> - **`toRefs`**：将一个响应式对象的所有属性转换为 `ref`。
> - **`toRef`**：将一个响应式对象的单个属性转换为 `ref`。
>
> 这两个方法在组合式 API 中非常有用，尤其是在处理响应式对象时，可以确保解构或传递的属性保持响应性。







### 计算属性

> ```vue
> let AllName = computed(() => {
>   return name.value + "----" + age.value;
> });
> 
> //这样定义的计算属性是 ==只读的== 不可修改
> ```
>
> 在 Vue 3 中，**`computed` 计算属性默认是只读的**，但是如果你需要修改它，可以使用 **带 `getter` 和 `setter` 的 `computed`**。
>
> 
>
> ```vue
> const fullName = computed({
>   get: () => name.value + " " + age.value, // 读取时返回拼接的字符串
>   set: (newValue) => {
>     const parts = newValue.split(" ");
>     name.value = parts[0]; // 修改 `name`
>     age.value = parseInt(parts[1]) || age.value; // 修改 `age`
>   }
> });
> ```
>
> 





### watch监视

> ## **`computed`（计算属性）**
>
> ### ✅ **适用于**：
>
> - 依赖**已有数据**，**基于它们派生新数据**。
> - 只有在**依赖的值发生变化**时才会重新计算，具有**缓存功能**。

> ## **`watch`（监听器）**
>
> ### ✅ **适用于**：
>
> - 监听数据的**变化**，然后**执行额外的副作用操作**（如：**发请求、手动修改变量、执行某些逻辑**）。
> - 适用于需要**执行异步操作**的情况（如 API 请求）。





> watch能监听==四种东西==
>
> 1、ref定义的数据
>
> 2、reactive定义的数据
>
> 3、函数返回的一个值
>
> 4、一个包含上述内容的数组





####  情况一

> ```vue
> import { reactive, watch } from "vue";
> 
> const person = reactive({ name: "张三", age: 18 });
> 
> watch(
>   () => person.age,
>   (newValue: number, oldValue: number) => {
>     console.log(`年龄变化了: ${oldValue} -> ${newValue}`);
>   }
> );
> 
> ```
>
> 

> 如何==停止==监视呢？
>
> ```vue
> import { ref, watch } from "vue";
> 
> const age = ref(18);
> 
> // 创建监听，并获取返回的停止函数
> const stopWatching = watch(age, (newValue, oldValue) => {
>   console.log(`年龄变化了: ${oldValue} -> ${newValue}`);
> });
> 
> // 3秒后停止监听
> setTimeout(() => {
>   console.log("停止监听 age");
>   stopWatching(); // 停止 watch
> }, 3000);
> 
> ```
>
> 



#### 情况二

> 监视【ref】定义的对象类型数据
>
> ```vue
> import { ref, watch } from "vue";
> 
> const user = ref({ name: "张三", age: 18 });
> 
> watch(user, (newValue, oldValue) => {
>   console.log("用户对象发生变化：", newValue, oldValue);
> });
> 
> // 默认情况下，watch 只能检测 user 这个对象的引用是否发生变化（例如 user.value = {}），但不会检测 user.value.age 或 user.value.name 变化。
> ```
>
> 注意：若想监视对象内部属性的变化，需开启深度监视“deep”

---

> ```vue
> watch(
>   user,
>   (newUser, oldUser) => {
>     console.log("用户数据变化：", newUser, oldUser);
>   },
>   { deep: true } // 深度监听对象内部所有属性
> );
> 
> ```
>
> 





> 监听一次停止
>
> ```vue
> const stopWatching = watch(
>   age,
>   (newValue, oldValue) => {
>     console.log(`age 变化了一次: ${oldValue} -> ${newValue}`);
>     stopWatching(); // 只执行一次后停止监听
>   },
>   { immediate: true } // 立即执行一次
> );
> 
> ```
>
> 如果你希望 `watch` **只监听一次变化后自动停止**，可以使用 `immediate` + `stopWatching`：
>
> 📌 **应用场景**：只需要监听某个值的**第一次变化**，后续不再监听。





#### 情况三

> 监视reactive定义的对象类型，且==默认==开启了深度监视





#### 情况四

> 监视 ref 或 reactive 定义的对象类型数据中的某个属性，注意：
>
> 1、若该属性值不是对象类型，需要写成函数形式
>
> 2、若该属性依然是对象类型，可直接编辑，也可写成函数，建议写成函数。



> 局部监视：基本类型（函数形式）
>
> ![image-20250302084259479](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20250302084259479.png)

> 局部监视：对象，可以直接监视
>
> ![image-20250302084712855](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20250302084712855.png)
>
> 但是此种改整个对象，不会触发watch

> 建议使用函数式：需要深度加上deep：true![image-20250302084925143](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20250302084925143.png)







#### 情况五

> 监视多种数据
>
> ![image-20250302085317595](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20250302085317595.png)







### watchEffect

> ```vue
> <script setup>
> import { ref, watchEffect } from 'vue'
> 
> const count = ref(0)
> 
> watchEffect(() => {
>   console.log(`count 的值变了：${count.value}`)
> })
> 
> // 修改 count，watchEffect 会触发回调
> setTimeout(() => {
>   count.value = 10 // 触发 watchEffect
> }, 1000)
> </script>
> 
> ```
>
> ## `watchEffect` vs `watch`
>
> | 特性         | `watchEffect`                                   | `watch`                          |
> | ------------ | ----------------------------------------------- | -------------------------------- |
> | **依赖收集** | **自动收集**                                    | **手动指定**                     |
> | **首次执行** | **立即执行**                                    | **不会立即执行**（默认）         |
> | **适用场景** | **处理副作用**（如 API 请求、日志、DOM 操作等） | **监听具体的 ref/reactive 变量** |
>
> ### `watch` 示例（需要手动指定监听的值）
>
> ##  适用场景
>
> **使用 `watchEffect` 的场景：**
>
> - **自动收集依赖的逻辑**，比如计算数据
> - **执行副作用操作**，比如打印日志、API 请求、操作 DOM
> - **监听多个响应式值**，不需要手动指定
>
> **使用 `watch` 更合适的场景：**
>
> - **监听单个变量的变化**
> - **需要访问旧值和新值**（`watchEffect` 无法获取 `oldValue`）





### 标签的ref属性

> 不建议项目里用id，会造成冲突
>
> ```vue
> <script setup>
> import { ref, onMounted } from 'vue'
> 
> const myDiv = ref(null) // 初始化 ref，默认是 null
> 
> onMounted(() => {
>   console.log(myDiv.value) // 获取 DOM 元素
>   myDiv.value.innerText = "Hello, Vue 3!"
> })
> </script>
> 
> <template>
>   <div ref="myDiv">我是一个 div</div>
> </template>
> 
> //ref 绑定 DOM 元素后，可以在 onMounted 钩子中访问它。
> ```
>
> **解释：**
>
> - `const myDiv = ref(null)` —— 创建一个 `ref` 变量用于存储 DOM 引用
> - `ref="myDiv"` —— 在模板中，将 `div` 绑定到 `myDiv`
> - `onMounted(() => console.log(myDiv.value))` —— 在 `onMounted` 钩子中访问这个 `div`（因为在 `setup` 中，DOM 还没挂载）
> - `myDiv.value.innerText = "Hello, Vue 3!"` —— 修改 `div` 的内容
>
> 
>
> 引入ref无非两种，普通html标签或者是vue组件







### ts语法

> ![image-20250302103438979](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20250302103438979.png)
>
> 进行了接口约束，约束了Person属性类型
>
> ![image-20250302103607395](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20250302103607395.png)





### 父子组件串字传值props

> ### Vue 的 `props` 用于在 **父组件** 向 **子组件** 传递数据。
>
> 在 Vue 3 的 `setup` 语法中，`props` 作为 `defineProps()` 的参数来使用。下面我们详细讲解 `props` 的用法，包括 **基础用法、类型校验、默认值、必填、动态 props** 等。
>
> ------
>
> ## 1️⃣ **基础用法**
>
> ### **父组件**
>
> ```vue
> <script setup>
> import Child from './Child.vue'
> </script>
> 
> <template>
>   <Child name="张三" age="25" />
> </template>
> ```
>
> ### **子组件（Child.vue）**
>
> ```vue
> <script setup>
> const props = defineProps(['name', 'age'])
> </script>
> 
> <template>
>   <p>姓名：{{ props.name }}</p>
>   <p>年龄：{{ props.age }}</p>
> </template>
> ```
>
> ✅ `defineProps()` 用于**声明 props**，接受一个数组或对象。
>  ✅ 在 `setup` 中，`props` 是 **响应式** 的，但不能直接修改。
>
> ------
>
> ## 2️⃣ **类型校验**
>
> 在 Vue 3 中，可以使用对象语法定义 `props`，并指定**数据类型**，如果传入的值类型不匹配，Vue 会在 **开发环境** 提示警告。
>
> ```vue
> <script setup>
> defineProps({
>   name: String,  // 传入字符串
>   age: Number,   // 传入数字
>   isAdmin: Boolean, // 传入布尔值
>   hobbies: Array,   // 传入数组
>   userInfo: Object  // 传入对象
> })
> </script>
> 
> <template>
>   <p>姓名：{{ name }}</p>
>   <p>年龄：{{ age }}</p>
>   <p>是否管理员：{{ isAdmin }}</p>
>   <p>爱好：{{ hobbies.join(', ') }}</p>
>   <p>用户信息：{{ userInfo.name }} - {{ userInfo.email }}</p>
> </template>
> ```
>
> ✅ **数据类型不匹配时**，Vue 在 **开发环境** 会报错，但不会影响运行。
>
> ------
>
> ## 3️⃣ **默认值**
>
> 如果 `props` 可能 **未传递**，可以通过 `default` 指定**默认值**。
>
> ```vue
> <script setup>
> defineProps({
>   name: { type: String, default: "匿名用户" },
>   age: { type: Number, default: 18 },
>   isAdmin: { type: Boolean, default: false }
> })
> </script>
> 
> <template>
>   <p>姓名：{{ name }}</p>
>   <p>年龄：{{ age }}</p>
>   <p>是否管理员：{{ isAdmin ? "是" : "否" }}</p>
> </template>
> ```
>
> ✅ `default` 只有当父组件 **没有传递** 该 prop 时，才会生效。
>
> ------
>
> ## 4️⃣ **必填校验**
>
> 如果 `props` 必须传递，可以使用 `required: true`。
>
> ```vue
> <script setup>
> defineProps({
>   name: { type: String, required: true }, // 必须传递
>   age: { type: Number, required: true }
> })
> </script>
> 
> <template>
>   <p>姓名：{{ name }}</p>
>   <p>年龄：{{ age }}</p>
> </template>
> ```
>
> ✅ 如果 `props` **未传递**，Vue 在 **开发环境** 会报错。
>
> ------
>
> ## 5️⃣ **动态传递 `props`**
>
> 如果 `props` 需要动态赋值，可以使用 `v-bind`：
>
> ### **父组件**
>
> ```vue
> <script setup>
> import Child from './Child.vue'
> 
> const user = {
>   name: "李四",
>   age: 30
> }
> </script>
> 
> <template>
>   <!-- 直接绑定对象 -->
>   <Child v-bind="user" />
> </template>
> ```
>
> ✅ `v-bind="user"` **会自动把 `user` 的所有属性作为 `props` 传入**。
>
> ------
>
> ## 6️⃣ **非响应式 `props` 变量**
>
> 在 `setup` 里，`props` **是响应式的，但不能修改**。
>
> ```vue
> <script setup>
> const props = defineProps(['name'])
> 
> // ❌ 不能直接修改 props
> // props.name = "新名字" // 这样会报错
> </script>
> ```
>
> 如果要修改 `props`，需要 **拷贝到一个 `ref` 或 `computed` 变量**。
>
> ```vue
> <script setup>
> import { ref, computed } from 'vue'
> 
> const props = defineProps(['name'])
> 
> // 使用 ref 进行拷贝
> const newName = ref(props.name)
> 
> // 使用 computed 进行派生
> const displayName = computed(() => props.name.toUpperCase())
> </script>
> 
> <template>
>   <p>原始 name: {{ props.name }}</p>
>   <p>可修改的 newName: {{ newName }}</p>
>   <p>转换后的 displayName: {{ displayName }}</p>
> </template>
> ```
>
> ✅ `props` 不能直接改，但可以 **拷贝到 `ref` 或 `computed`** 进行使用。
>
> ------
>
> ## 7️⃣ **Prop 的 `default` 也可以是函数**
>
> 有时候 `default` 值需要是一个对象或数组，可以使用**函数**返回。
>
> ```vue
> <script setup>
> defineProps({
>   user: {
>     type: Object,
>     default: () => ({ name: "默认用户", age: 20 })
>   }
> })
> </script>
> 
> <template>
>   <p>姓名：{{ user.name }}</p>
>   <p>年龄：{{ user.age }}</p>
> </template>
> ```
>
> ✅ **如果 `default` 直接是对象/数组，会被所有实例共享，可能会造成数据污染**，所以必须使用 `()=>({})` **返回新对象**。
>
> ------
>
> ## 8️⃣ **子组件向父组件传递事件**
>
> 有时候，子组件需要**通知父组件**，可以用 `emit` 事件：
>
> ### **子组件（Child.vue）**
>
> ```vue
> <script setup>
> const props = defineProps(['name'])
> const emit = defineEmits(['updateName']) // 定义事件
> 
> const changeName = () => {
>   emit('updateName', "新名字") // 触发事件，并传递新值
> }
> </script>
> 
> <template>
>   <p>姓名：{{ name }}</p>
>   <button @click="changeName">修改姓名</button>
> </template>
> ```
>
> ### **父组件**
>
> ```vue
> <script setup>
> import { ref } from 'vue'
> import Child from './Child.vue'
> 
> const userName = ref("张三")
> 
> const handleUpdateName = (newName) => {
>   userName.value = newName
> }
> </script>
> 
> <template>
>   <p>父组件的 name: {{ userName }}</p>
>   <Child :name="userName" @updateName="handleUpdateName" />
> </template>
> ```
>
> ✅ **子组件用 `emit('updateName', 值)` 触发事件**，父组件用 `@updateName="handleUpdateName"` 监听。
>
> ------
>
> ## 9️⃣ **`v-model` 绑定 `props`**
>
> Vue 3 **默认支持 `v-model` 绑定 props**，不需要手动定义 `emit` 事件。
>
> ### **子组件**
>
> ```vue
> <script setup>
> defineProps(['modelValue'])
> defineEmits(['update:modelValue'])
> </script>
> 
> <template>
>   <input :value="modelValue" @input="$emit('update:modelValue', $event.target.value)" />
> </template>
> ```
>
> ### **父组件**
>
> ```vue
> <Child v-model="name" />
> ```
>
> ✅ Vue 3 里，`v-model` **自动绑定 `modelValue` 这个 `prop`**，也会触发 `update:modelValue` 事件。
>
> ------
>
> ## 🔟 **总结**
>
> ✅ `props` 用于 **父组件向子组件传递数据**
>  ✅ 不能修改 `props`，需要 **拷贝到 `ref` 变量**
>  ✅ `props` 可以 **类型校验、默认值、必填**
>  ✅ `v-bind` 可以 **动态传递 `props`**
>  ✅ `emit` 可以 **让子组件通知父组件**
>  ✅ `v-model` 自动绑定 `modelValue

> 在 Vue 3 里，**`:` (v-bind) 必须加在 `props` 绑定的变量前**，否则 Vue 会把它当作**普通字符串**处理
>
> 
>
> ```vue
> <Child :users="users" />
> 
> ```
>
> 





### 生命周期

#### Vue2的生命周期

> ##  **生命周期适用场景**
>
> | 钩子            | 适用场景                                     |
> | --------------- | -------------------------------------------- |
> | `beforeCreate`  | 初始化之前，不能使用 `data`                  |
> | `created`       | 可以访问 `data`，但不能操作 DOM              |
> | `beforeMount`   | 模板编译完成，DOM 还没渲染                   |
> | `mounted`       | **DOM 操作最佳时机**（如获取元素、事件监听） |
> | `beforeUpdate`  | 数据变了，但 **DOM 还没更新**                |
> | `updated`       | 数据变了，DOM **已更新**                     |
> | `beforeDestroy` | **清理定时器、事件监听、解绑 WebSocket**     |
> | `destroyed`     | 组件彻底销毁，数据、事件都清空               |
>
> ------
>
> ## **📌  总结**
>
> ✅ **Vue 2 生命周期大致顺序：**
>
> 1. `beforeCreate` → `created` （数据初始化）
> 2. `beforeMount` → `mounted` （DOM 挂载）
> 3. `beforeUpdate` → `updated` （数据更新）
> 4. `beforeDestroy` → `destroyed` （组件销毁）
>
> 🚀 **一般用 `mounted` 处理 DOM，用 `beforeDestroy` 清理事件！**







#### Vue3的生命周期

> ##  **Vue 3 组合式 API 生命周期钩子**
>
> Vue 3 提供了一系列 `onXxx` 形式的生命周期函数，适用于 **`setup()`** 语法：
>
> | Vue 2 选项式    | Vue 3 组合式 API  | 触发时机                        |
> | --------------- | ----------------- | ------------------------------- |
> | `beforeCreate`  | ❌ **(废弃)**      | 组件创建前，不能访问 `data`     |
> | `created`       | ❌ **(废弃)**      | 组件创建完成，可访问 `data`     |
> | `beforeMount`   | `onBeforeMount`   | 组件挂载前，DOM 未渲染          |
> | `mounted`       | `onMounted`       | 组件挂载完成，可访问 `this.$el` |
> | `beforeUpdate`  | `onBeforeUpdate`  | 组件更新前，DOM 仍是旧的        |
> | `updated`       | `onUpdated`       | 组件更新完成，DOM 已更新        |
> | `beforeDestroy` | `onBeforeUnmount` | 组件销毁前，适合清理副作用      |
> | `destroyed`     | `onUnmounted`     | 组件销毁完成，DOM 彻底清空      |
>
> 🚀 **Vue 3 推荐使用 `onXxx` 形式的组合式 API，而不是选项式 API 的生命周期！**





> ###  **组合式 API 版（推荐写法）**
>
> ```
> <template>
>   <div>{{ message }}</div>
> </template>
> 
> <script setup>
> import { ref, onMounted, onBeforeUnmount } from "vue";
> 
> const message = ref("Hello Vue 3!");
> 
> // 组件挂载完成后执行
> onMounted(() => {
>   console.log("组件已挂载");
> });
> 
> // 组件销毁前执行
> onBeforeUnmount(() => {
>   console.log("组件即将销毁");
> });
> </script>
> ```





> 父和子的生命周期：
>
> ### **父子组件生命周期的关系**
>
> Vue 的生命周期是自下而上的。也就是说，**子组件的生命周期比父组件的生命周期早**，并且在父组件的生命周期结束时，子组件的生命周期会结束。







### hooks

> 安装axios 调用http请求
>
> 就是拆解需求，封装在一起。
>
> ###  **Vue 中的 Composition API 类似于 Hooks**
>
> Vue 的 **Composition API** 提供了一些类似于 React Hooks 的功能，帮助我们在 Vue 函数组件中更好地组织和复用逻辑，主要使用 `setup()` 函数来管理逻辑。
>
> #### **Vue 中的 Composition API 示例：**
>
> ```
> <template>
>   <div>
>     <p>当前计数: {{ count }}</p>
>     <button @click="increment">增加</button>
>   </div>
> </template>
> 
> <script>
> import { ref, onMounted } from 'vue';
> 
> export default {
>   setup() {
>     const count = ref(0);  // ref 用于声明响应式状态
> 
>     // 使用生命周期钩子 onMounted
>     onMounted(() => {
>       console.log('组件已挂载');
>       fetchData();
>     });
> 
>     const increment = () => {
>       count.value += 1;
>     };
> 
>     const fetchData = async () => {
>       const response = await fetch('https://jsonplaceholder.typicode.com/posts');
>       const data = await response.json();
>       console.log(data);
>     };
> 
>     return { count, increment };
>   }
> };
> </script>
> ```







### 路由

> npm i vue-router
>
> 在 Vue 3 项目中，通常使用 Vue Router 来进行页面导航。一个常见的路由配置文件是 `index.ts`，通常放在 `src/router/` 目录下。
>
> 下面是一个基本的 Vue 3 路由配置文件 `index.ts` 示例：
>
> ### **`src/router/index.ts`**
>
> ```typescript
> import { createRouter, createWebHistory, RouteRecordRaw } from 'vue-router';
> import Home from '../views/Home.vue';
> import About from '../views/About.vue';
> import NotFound from '../views/NotFound.vue';
> 
> // 路由配置
> const routes: Array<RouteRecordRaw> = [
>   {
>     path: '/',
>     name: 'Home',
>     component: Home,
>   },
>   {
>     path: '/about',
>     name: 'About',
>     component: About,
>   },
>   // 处理未找到页面的路由
>   {
>     path: '/:pathMatch(.*)*', // 匹配所有路径
>     name: 'NotFound',
>     component: NotFound,
>   },
> ];
> 
> // 创建路由实例
> const router = createRouter({
>   history: createWebHistory(import.meta.env.BASE_URL), // 使用 HTML5 History 模式
>   routes, // 路由配置
> });
> 
> export default router;
> ```
>
> ### **解释：**
>
> 1. **`import { createRouter, createWebHistory } from 'vue-router';`**
>     引入 Vue Router 的核心函数 `createRouter` 和 `createWebHistory`。
> 2. **`const routes: Array<RouteRecordRaw> = [...]`**
>     定义路由配置，`routes` 是一个数组，包含每个路由的路径、名称和组件。每个路由的配置项可以包括：
>    - `path`: 路径，用户访问的 URL。
>    - `name`: 路由名称，便于导航。
>    - `component`: 该路径对应的 Vue 组件。
> 3. **`createWebHistory(import.meta.env.BASE_URL)`**
>     `createWebHistory` 用于启用 HTML5 的 History 模式，这意味着 URL 中不会带有 `#` 符号，像传统的 SPA 应用一样，支持浏览器的前进、后退按钮。
> 4. **动态路径匹配：**
>     在最后一个路由配置中，`path: '/:pathMatch(.*)*'` 用于匹配任何未定义的路径，这通常用于处理 404 页面（页面未找到）。
>
> ### **在主应用中使用路由**
>
> 在你的 Vue 组件中使用路由，通常是在 `main.ts` 或 `main.js` 文件中进行配置：
>
> #### **`src/main.ts`**
>
> ```typescript
> import { createApp } from 'vue';
> import App from './App.vue';
> import router from './router'; // 导入路由
> 
> const app = createApp(App);
> app.use(router); // 使用路由
> app.mount('#app');
> ```
>
> ### **如何添加新的路由：**
>
> - 当你需要添加新页面时，只需创建一个新的 Vue 组件（例如 `Contact.vue`），然后在 `index.ts` 的 `routes` 数组中添加一个新的路由配置：
>
> ```typescript
> import Contact from '../views/Contact.vue';
> 
> const routes: Array<RouteRecordRaw> = [
>   {
>     path: '/',
>     name: 'Home',
>     component: Home,
>   },
>   {
>     path: '/about',
>     name: 'About',
>     component: About,
>   },
>   {
>     path: '/contact',
>     name: 'Contact',
>     component: Contact,  // 新增的路由
>   },
>   {
>     path: '/:pathMatch(.*)*',
>     name: 'NotFound',
>     component: NotFound,
>   },
> ];
> ```
>
> 这就是一个基本的 Vue 3 路由配置文件，它处理页面导航并管理视图组件的加载。如果你的项目较复杂，可能还需要考虑懒加载、路由守卫等内容，但这个基础配置应该足以让你开始使用 Vue Router 了。



#### 路由组件和一般组件

> 一般组件 </TestVue>
>
> 路由组件：靠路由规则渲染出来的
>
> components——一般放组件
>
> pages/views——一般放视图
>
> 
>
> **路由切换**会导致与该路由相关的组件的生命周期钩子函数的触发。这是因为在 Vue 中，每当路由发生变化时，相关的组件（即与新的路由路径匹配的组件）会被重新渲染，从而触发生命周期钩子。



#### 路由工作模式：hash模式和history模式

> history：不带#号，美观一些
>
> vue2：mode：‘history’
>
> vue3：history：createWebHistory()
>
> 缺点：刷新会有404错误
>
> 
>
> 
>
> hash
>
> 缺点：seo优化很差





> ### **总结：**
>
> | **特性**           | **Hash 模式**                      | **History 模式**                   |
> | ------------------ | ---------------------------------- | ---------------------------------- |
> | **URL 结构**       | `http://example.com/#/home`        | `http://example.com/home`          |
> | **浏览器兼容性**   | 兼容所有浏览器，包括旧版浏览器     | 只支持现代浏览器（IE10+）          |
> | **页面刷新**       | 不会触发页面刷新                   | 可能会触发页面刷新，依赖服务器配置 |
> | **服务器配置要求** | 不需要特别配置                     | 需要服务器支持，返回 `index.html`  |
> | **SEO 支持**       | 不太友好，搜索引擎不能索引哈希路径 | 较好，路径直接可以被搜索引擎抓取   |
> | **实现复杂度**     | 实现简单，配置少                   | 需要配置服务器                     |
>
> ### **选择哪个模式？**
>
> - **使用 `hash` 模式：** 如果你的项目需要兼容旧版浏览器，或者你不想处理服务器配置问题（例如没有控制服务器的情况），`hash` 模式是一个较为简单且兼容性强的选择。
> - **使用 `history` 模式：** 如果你追求更干净、简洁的 URL 结构，并且项目只面向现代浏览器，`history` 模式会是一个更好的选择，尤其对于需要良好 SEO 支持的单页面应用（SPA）来说。







#### 嵌套路由

> 在 Vue Router 中，**嵌套路由**是指在一个路由的组件内部嵌套另一个路由。这使得你能够构建具有多级结构的页面，并且每个页面的不同部分可以由不同的组件来渲染。嵌套路由在开发具有多层页面结构或父子组件关系的应用时非常有用。
>
> ### **嵌套路由的基本概念**
>
> 嵌套路由就是在父路由的组件中定义子路由（也就是嵌套的路由）。在父组件中，会有一个 `<router-view>` 元素用于显示当前路由的组件，而子路由的组件会在该 `<router-view>` 中渲染。
>
> ### **示例：基础嵌套路由**
>
> 假设我们有一个父路由 `Home`，它有两个子路由：`Dashboard` 和 `Profile`。
>
> #### **1. 设置路由**
>
> 首先，在 `router/index.ts` 中定义路由配置，父路由 `Home` 中嵌套两个子路由 `Dashboard` 和 `Profile`。
>
> ```typescript
> // router/index.ts
> import { createRouter, createWebHistory } from 'vue-router';
> import Home from '../views/Home.vue';
> import Dashboard from '../views/Dashboard.vue';
> import Profile from '../views/Profile.vue';
> 
> const routes = [
>   {
>     path: '/',
>     name: 'Home',
>     component: Home,
>     children: [
>       {
>         path: 'dashboard',
>         name: 'Dashboard',
>         component: Dashboard
>       },
>       {
>         path: 'profile',
>         name: 'Profile',
>         component: Profile
>       }
>     ]
>   }
> ];
> 
> const router = createRouter({
>   history: createWebHistory(),
>   routes
> });
> 
> export default router;
> ```
>
> 在这里，`Home` 组件是父路由，它有两个子路由：`Dashboard` 和 `Profile`。
>
> #### **2. 配置父组件的 `<router-view>`**
>
> 接下来，在 `Home.vue` 组件中，我们需要插入一个 `<router-view>` 来显示匹配的子组件。
>
> ```vue
> <!-- src/views/Home.vue -->
> <template>
>   <div>
>     <h1>Welcome to Home</h1>
>     <nav>
>       <router-link to="/dashboard">Dashboard</router-link>
>       <router-link to="/profile">Profile</router-link>
>     </nav>
>     <!-- 子路由将会渲染到这里 -->
>     <router-view></router-view>
>   </div>
> </template>
> 
> <script>
> export default {
>   name: 'Home',
> };
> </script>
> ```
>
> `<router-view>` 是一个占位符，Vue Router 会将当前匹配的子组件渲染到这个位置。比如，当用户访问 `/dashboard` 时，`Dashboard` 组件会显示在这个 `<router-view>` 中；当用户访问 `/profile` 时，`Profile` 组件会显示在这里。
>
> #### **3. 配置子组件（`Dashboard.vue` 和 `Profile.vue`）**
>
> 然后，创建 `Dashboard.vue` 和 `Profile.vue` 组件：
>
> ```vue
> <!-- src/views/Dashboard.vue -->
> <template>
>   <div>
>     <h2>Dashboard</h2>
>     <p>Welcome to your dashboard.</p>
>   </div>
> </template>
> 
> <script>
> export default {
>   name: 'Dashboard'
> };
> </script>
> <!-- src/views/Profile.vue -->
> <template>
>   <div>
>     <h2>Profile</h2>
>     <p>Your profile information goes here.</p>
>   </div>
> </template>
> 
> <script>
> export default {
>   name: 'Profile'
> };
> </script>
> ```
>
> ### **路由切换效果**
>
> - 当用户访问 `/`（父路由）时，`Home` 组件会被渲染，并显示 `Dashboard` 和 `Profile` 的链接。
> - 当用户点击 `Dashboard` 链接时，URL 会变成 `/dashboard`，并且 `Dashboard` 组件会显示在 `Home` 组件的 `<router-view>` 中。
> - 当用户点击 `Profile` 链接时，URL 会变成 `/profile`，并且 `Profile` 组件会显示在 `Home` 组件的 `<router-view>` 中。
>
> ### **嵌套路径中的 `:param`（路由参数）**
>
> 嵌套路由也支持动态路由参数。如果你希望在子路由中使用路由参数，可以像普通路由一样使用 `:param`，并在子组件中通过 `this.$route.params` 获取它。
>
> 例如，在 `Profile` 组件中，我们可以添加一个动态路由参数 `userId`：
>
> ```typescript
> {
>   path: 'profile/:userId',
>   name: 'Profile',
>   component: Profile
> }
> ```
>
> 然后，访问 `/profile/123` 时，`Profile` 组件会收到 `userId` 的值（即 `123`）。你可以在 `Profile.vue` 中访问这个参数：
>
> ```vue
> <!-- src/views/Profile.vue -->
> <template>
>   <div>
>     <h2>Profile of User {{ userId }}</h2>
>   </div>
> </template>
> 
> <script>
> export default {
>   name: 'Profile',
>   computed: {
>     userId() {
>       return this.$route.params.userId;
>     }
>   }
> };
> </script>
> ```
>
> ### **命名视图与嵌套路由**
>
> 除了使用默认的 `<router-view>`，Vue Router 还支持命名视图，这使得我们可以在同一父组件中定义多个嵌套路由，每个子路由渲染到不同的位置。
>
> #### **1. 设置命名视图**
>
> 在父组件中，使用不同的 `<router-view>` 来对应不同的视图。例如，在 `Home` 组件中，可以使用两个不同的视图来分别显示 `Sidebar` 和 `MainContent`：
>
> ```vue
> <!-- src/views/Home.vue -->
> <template>
>   <div>
>     <h1>Welcome to Home</h1>
>     <div class="layout">
>       <!-- 显示 Sidebar -->
>       <router-view name="sidebar"></router-view>
>       
>       <!-- 显示 Main Content -->
>       <router-view></router-view>
>     </div>
>   </div>
> </template>
> 
> <script>
> export default {
>   name: 'Home'
> };
> </script>
> ```
>
> #### **2. 配置路由**
>
> 在路由配置中，可以指定每个子路由要渲染的命名视图：
>
> ```typescript
> {
>   path: '/',
>   name: 'Home',
>   component: Home,
>   children: [
>     {
>       path: '',
>       components: {
>         default: MainContent,
>         sidebar: Sidebar
>       }
>     }
>   ]
> }
> ```
>
> ### **总结**
>
> - **父子路由结构**：父路由可以嵌套多个子路由，父组件通过 `<router-view>` 来渲染子路由。
> - **动态参数**：子路由也可以支持动态路由参数，父路由的参数可以传递给子路由。
> - **命名视图**：通过命名视图，可以在同一个父组件中显示多个子组件。
>
> 嵌套路由使得 Vue 应用能够实现复杂的页面结构，每个页面的不同部分由不同的组件来渲染，非常适合大型应用和多层级页面的设计。







#### 路由的query 和 params

> 在 Vue Router 中，**`query`** 和 **`params`** 都是用来传递和访问路由信息的方式，但它们的使用方式和场景有所不同。
>
> ### **1. 路由 `params`**
>
> `params` 是路由路径的一部分，通常用于动态路由，传递 URL 中的路径参数。
>
> #### **特点**：
>
> - `params` 是 URL 路径的一部分。
> - `params` 用于在路径中定义动态参数（也叫路由参数）。
> - 它们通过定义路由时的 `:paramName` 来指定。
>
> #### **如何使用 `params`**：
>
> 例如，我们有一个路由 `/user/:userId`，其中 `userId` 就是一个动态的路由参数。
>
> ```typescript
> // router/index.ts
> const routes = [
>   {
>     path: '/user/:userId',
>     name: 'User',
>     component: User
>   }
> ];
> ```
>
> 在这个路由中，`userId` 就是一个动态路由参数。
>
> - 访问 `/user/123`，`userId` 的值是 `123`。
> - 在组件中访问该参数：
>
> ```javascript
> // 在 User.vue 组件中
> export default {
>   computed: {
>     userId() {
>       return this.$route.params.userId;  // 获取路由参数 userId
>     }
>   }
> }
> ```
>
> #### **`params` 的特点**：
>
> - 仅适用于路径中的动态部分。
> - 是 URL 的一部分，因此不会改变查询字符串。
>
> #### **`params` 的使用场景**：
>
> - 用于确定资源的唯一标识符，例如用户 ID、文章 ID 等。
>
> ### **2. 路由 `query`**
>
> `query` 是 URL 查询字符串的一部分，它通常用于传递较为灵活的参数，可以用于过滤、分页、排序等操作。
>
> #### **特点**：
>
> - `query` 是 URL 中 `?` 后面的一部分，参数通过 `&` 连接。
> - 适用于非必需的查询参数。
> - `query` 的参数是通过 URL 的查询字符串传递的，如 `?key=value`。
>
> #### **如何使用 `query`**：
>
> 例如，我们有一个路由 `/search`，并且希望通过查询字符串传递搜索关键字。
>
> ```typescript
> // router/index.ts
> const routes = [
>   {
>     path: '/search',
>     name: 'Search',
>     component: Search
>   }
> ];
> ```
>
> 然后，我们可以通过 URL `/search?query=vue&sort=desc` 来传递查询参数。
>
> - `query` 参数可以通过 `$route.query` 获取：
>
> ```javascript
> // 在 Search.vue 组件中
> export default {
>   computed: {
>     searchQuery() {
>       return this.$route.query.query;  // 获取 query 参数 query
>     },
>     sortOrder() {
>       return this.$route.query.sort;  // 获取 query 参数 sort
>     }
>   }
> }
> ```
>
> - 对于 `/search?query=vue&sort=desc`，`this.$route.query.query` 将返回 `'vue'`，而 `this.$route.query.sort` 将返回 `'desc'`。
>
> #### **`query` 的特点**：
>
> - 是 URL 查询字符串的一部分。
> - 不会影响路径的结构。
> - 参数是可选的，不影响路由匹配。
>
> #### **`query` 的使用场景**：
>
> - 用于传递可选的、非路径相关的参数，例如过滤条件、排序方式、分页信息等。
>
> ### **`params` 和 `query` 的区别**
>
> | **特性**             | **`params`**                        | **`query`**                             |
> | -------------------- | ----------------------------------- | --------------------------------------- |
> | **传递方式**         | 通过 URL 路径传递                   | 通过查询字符串传递                      |
> | **定义位置**         | URL 路径的一部分，如 `/user/:id`    | URL 查询字符串的一部分，如 `?key=value` |
> | **访问方式**         | 通过 `this.$route.params` 访问      | 通过 `this.$route.query` 访问           |
> | **是否影响路由匹配** | 会影响路由匹配                      | 不会影响路由匹配                        |
> | **是否可选**         | 必须在路径中提供                    | 可选                                    |
> | **适用场景**         | 资源标识符、必需的动态参数（如 ID） | 过滤条件、分页、排序等可选参数          |
>
> ### **示例：结合 `params` 和 `query` 使用**
>
> 假设你有一个用户详情页面，路径是 `/user/:userId`，并且你想要根据查询字符串中的 `showPosts` 参数来决定是否显示用户的帖子列表。
>
> - 路由定义：
>
> ```typescript
> const routes = [
>   {
>     path: '/user/:userId',
>     name: 'User',
>     component: User
>   }
> ];
> ```
>
> - 在 URL 中访问 `/user/123?showPosts=true`，我们可以通过 `params` 和 `query` 组合来获取相应的数据：
>
> ```javascript
> // 在 User.vue 组件中
> export default {
>   computed: {
>     userId() {
>       return this.$route.params.userId;  // 获取 userId
>     },
>     showPosts() {
>       return this.$route.query.showPosts === 'true';  // 获取 query 参数 showPosts
>     }
>   }
> }
> ```
>
> 这样，你就可以在组件中根据 `userId` 来显示特定用户的信息，同时根据 `query` 参数来控制是否显示用户的帖子列表。
>
> ### **总结**
>
> - **`params`**：用于定义 URL 路径中的动态部分（例如 `/user/:id`），适合传递必需的、标识性强的参数。
> - **`query`**：用于传递 URL 中的查询字符串（例如 `/search?query=vue`），适合传递可选参数，如过滤条件、分页信息等。
>
> 这两种方式可以结合使用，根据具体需求选择合适的方式来传递数据。





#### 路由的props配置

> ## **`props` 的三种用法**
>
> Vue Router 提供了 **三种方式** 来配置 `props` 选项：
>
> ### **① 布尔模式 (`props: true`)**
>
> 这种方式会把 `params` 作为 `props` 传递给组件。
>
> #### **示例**
>
> ```
> typescript复制编辑const routes = [
>   {
>     path: '/user/:id',
>     component: User,
>     props: true  // 让 `id` 变成组件的 prop
>   }
> ];
> ```
>
> **在 `User.vue` 组件中：**
>
> ```
> vue复制编辑<script setup>
> defineProps(['id'])
> </script>
> 
> <template>
>   <div>用户 ID: {{ id }}</div>
> </template>
> ```
>
> ✅ **优点：**
>
> - 组件不需要依赖 `this.$route` 或 `useRoute()`，更加清晰独立。
> - 适用于**只使用 `params` 传递参数**的情况。
>
> ⚠️ **限制：**
>
> - **只能传递 `params`**，如果 `query` 或 `hash` 需要传递，就要用下面的模式。
>
> ------
>
> ### **② 对象模式 (`props: {}`)**
>
> 可以手动指定固定的 `props` 值，而不是从 `route` 里动态获取。
>
> #### **示例**
>
> ```
> typescript复制编辑const routes = [
>   {
>     path: '/about',
>     component: About,
>     props: { title: '关于我们' }  // 固定传递 title 属性
>   }
> ];
> ```
>
> **在 `About.vue` 组件中：**
>
> ```
> vue复制编辑<script setup>
> defineProps(['title'])
> </script>
> 
> <template>
>   <h2>{{ title }}</h2>
> </template>
> ```
>
> ✅ **适用场景：**
>
> - 组件接收的是**固定参数**，不需要从路由动态传递。
>
> ------
>
> ### **③ 函数模式 (`props: (route) => ({ ... })`)**
>
> 可以**自定义**如何从 `route` 里提取参数，包括 `params`、`query`、`hash`，甚至可以修改值。
>
> #### **示例**
>
> ```
> typescript复制编辑const routes = [
>   {
>     path: '/user/:id',
>     component: User,
>     props: route => ({ id: Number(route.params.id), keyword: route.query.keyword })
>   }
> ];
> ```
>
> **在 `User.vue` 组件中：**
>
> ```
> vue复制编辑<script setup>
> defineProps(['id', 'keyword'])
> </script>
> 
> <template>
>   <div>用户 ID: {{ id }}</div>
>   <div>搜索关键词: {{ keyword }}</div>
> </template>
> ```
>
> ✅ **适用场景：**
>
> - **可以传递 `params`、`query` 和 `hash`**。
> - **可以对参数进行转换**（如上例中 `id` 变成 `Number`）。
> - **可以提供默认值**（例如 `keyword || '默认值'`）。
>
> ------
>
> ## **3. `props` 三种模式对比**
>
> | 方式                          | 适用场景                             | 例子                                                |
> | ----------------------------- | ------------------------------------ | --------------------------------------------------- |
> | `props: true`                 | 适用于 `params` 传递                 | `/user/:id` 组件能直接接收 `id` 作为 `props`        |
> | `props: {}`                   | 适用于固定参数                       | `props: { title: '关于我们' }` 组件固定接收 `title` |
> | `props: (route) => ({ ... })` | 适用于 `params` + `query` + 处理数据 | 传递 `params.id` 和 `query.keyword` 并转换数据      |
>
> ------
>
> ## **4. 为什么使用 `props` 而不是 `this.$route`？**
>
> - **使组件更易复用**：组件可以在不依赖 `vue-router` 的情况下使用。
> - **更清晰的 API**：直接 `defineProps` 传入参数，而不是 `useRoute()` 获取参数。
> - **便于测试**：可以在测试中直接传递 `props`，不需要创建 Vue Router 实例。
>
> ------
>
> ## **总结**
>
> 1. `props: true` 适用于 **动态 `params`** 传递参数（`/user/:id`）。
> 2. `props: {}` 适用于 **固定参数** 传递（`props: { title: 'xxx' }`）。
> 3. `props: (route) => ({ ... })` 适用于 **动态 `params` + `query` 并进行处理**（如转换数据类型）。
>
> 在实际开发中，推荐使用 `props`，使组件更加独立、复用性更高，避免依赖 `$route`。





#### 路由跳转

> ### **`push`（默认行为，会形成历史记录）**
>
> - **每次跳转都会添加一条新的历史记录**，用户可以使用浏览器 **返回（back）** 按钮回到上一个页面。
> - 适用于需要保留历史记录的场景，比如从主页进入详情页后，用户可以返回到主页。
>
> #### **示例**
>
> ```
> ts
> 
> 
> 复制编辑
> router.push('/home');
> ```
>
> 或者：
>
> ```
> ts
> 
> 
> 复制编辑
> router.push({ path: '/user', query: { id: 123 } });
> ```
>
> ✅ **跳转后可以使用浏览器“后退”按钮返回上一个页面。**
>
> ------
>
> ### **2. `replace`（不会形成历史记录）**
>
> - **当前 URL 会被替换，不会形成新的历史记录**，用户无法通过浏览器“后退”按钮回到前一个页面。
> - 适用于 **重定向** 或 **不希望用户返回上一个页面** 的情况，比如登录后直接跳转到首页。
>
> #### **示例**
>
> ```
> ts
> 
> 
> 复制编辑
> router.replace('/home');
> ```
>
> 或者：
>
> ```
> ts
> 
> 
> 复制编辑
> router.replace({ path: '/user', query: { id: 123 } });
> ```
>
> ✅ **跳转后“后退”按钮不会回到上一个页面，而是回到更早的历史记录。**
>
> ------
>
> ### **3. `go(n)`（基于历史记录前进/后退）**
>
> - `go(n)` 可以在历史记录中 **前进（正数）或后退（负数）**。
> - 相当于 `window.history.go(n)`。
>
> #### **示例**
>
> ```
> ts复制编辑router.go(-1); // 后退一步
> router.go(1);  // 前进一步
> ```
>
> ------
>
> ### **4. `push` 和 `replace` 的区别**
>
> | 方式               | 是否形成历史记录 | 是否能返回上一个页面 |
> | ------------------ | ---------------- | -------------------- |
> | `router.push()`    | ✅ 形成历史记录   | ✅ 可以返回           |
> | `router.replace()` | ❌ 不形成历史记录 | ❌ 不能返回           |
>
> ------
>
> ### **5. `push` 和 `replace` 使用场景**
>
> | 场景                                   | 推荐使用           |
> | -------------------------------------- | ------------------ |
> | 进入详情页、普通跳转                   | `router.push()`    |
> | 登录后跳转首页、重定向                 | `router.replace()` |
> | 多步表单的下一步                       | `router.push()`    |
> | 表单提交后跳转（不希望用户回到表单页） | `router.replace()` |
>
> ------
>
> ### **6. 总结**
>
> - **默认情况下，Vue Router 使用 `push()`，会形成历史记录**。
> - **使用 `replace()` 不会形成历史记录**，适用于登录、重定向等场景。
> - **使用 `go(n)` 可控制前进或后退**。
>
> 在实际开发中，根据需求选择合适的跳转方式，避免不必要的历史记录或者让用户能够正确返回上一个页面。



#### 编程式路由导航

> 在 Vue 3 中，**编程式路由导航**（Programmatic Navigation）是指使用 `router.push()`、`router.replace()` 等方式 **在 JavaScript 代码中跳转路由**，而不是使用 `<router-link>` 组件。
>
> Vue Router 提供了 `useRouter()` 钩子（Vue 3 Composition API）或 `this.$router`（Vue 2 选项式 API）来进行编程式导航。
>
> ------
>
> ## **1. `push()`（添加历史记录）**
>
> `push()` 用于**跳转到新页面，并形成历史记录**，用户可以通过“返回”按钮回到上一个页面。
>
> ### **示例**
>
> ```ts
> import { useRouter } from 'vue-router';
> 
> const router = useRouter();
> 
> // 跳转到首页
> router.push('/home');
> 
> // 传递参数（params）
> router.push({ name: 'user', params: { id: 123 } });
> 
> // 传递查询参数（query）
> router.push({ path: '/search', query: { keyword: 'vue' } });
> ```
>
> ✅ **会形成历史记录，用户可以返回上一个页面。**
>
> ------
>
> ## **2. `replace()`（不添加历史记录）**
>
> `replace()` **不会形成新的历史记录**，而是**替换当前页面**，用户无法通过“返回”按钮回到前一个页面。
>
> ### **示例**
>
> ```ts
> router.replace('/dashboard');
> 
> // 传递参数（query）
> router.replace({ path: '/profile', query: { user: 'admin' } });
> ```
>
> ✅ **适用于重定向，避免用户返回到前一个页面。**
>
> ------
>
> ## **3. `go(n)`（前进/后退）**
>
> - `router.go(1)`：前进 1 步（相当于浏览器 `history.forward()`）。
> - `router.go(-1)`：后退 1 步（相当于浏览器 `history.back()`）。
>
> ### **示例**
>
> ```ts
> router.go(-1); // 后退
> router.go(2);  // 前进两步
> ```
>
> ------
>
> ## **4. `beforeEach` 守卫（跳转前拦截）**
>
> 可以使用 **导航守卫** 在路由跳转前执行一些逻辑，比如**权限验证、登录检查**等。
>
> ### **示例**
>
> ```ts
> import { createRouter, createWebHistory } from 'vue-router';
> 
> const router = createRouter({
>   history: createWebHistory(),
>   routes: [
>     { path: '/', component: () => import('@/views/Home.vue') },
>     { path: '/login', component: () => import('@/views/Login.vue') },
>     { path: '/dashboard', component: () => import('@/views/Dashboard.vue') },
>   ]
> });
> 
> // 在路由跳转前执行
> router.beforeEach((to, from, next) => {
>   const isAuthenticated = false; // 假设未登录
>   if (to.path === '/dashboard' && !isAuthenticated) {
>     next('/login'); // 强制跳转到登录页
>   } else {
>     next(); // 继续跳转
>   }
> });
> 
> export default router;
> ```
>
> ------
>
> ## **5. 在 Vue 组件中使用编程式导航**
>
> ### **① Composition API（Vue 3）**
>
> ```vue
> <script setup>
> import { useRouter } from 'vue-router';
> 
> const router = useRouter();
> 
> const goToProfile = () => {
>   router.push({ name: 'profile', params: { userId: 123 } });
> };
> </script>
> 
> <template>
>   <button @click="goToProfile">查看个人资料</button>
> </template>
> ```
>
> ### **② 选项式 API（Vue 2）**
>
> ```vue
> <script>
> export default {
>   methods: {
>     goToHome() {
>       this.$router.push('/home');
>     }
>   }
> }
> </script>
> 
> <template>
>   <button @click="goToHome">返回首页</button>
> </template>
> ```
>
> ------
>
> ## **6. 总结**
>
> | 方法        | 作用           | 是否形成历史记录 |
> | ----------- | -------------- | ---------------- |
> | `push()`    | 跳转到新页面   | ✅ 是             |
> | `replace()` | 替换当前页面   | ❌ 否             |
> | `go(n)`     | 前进/后退 n 步 | ❌ 取决于 n       |
>
> 📌 **建议**：
>
> - **`push()`** 适用于普通导航（如进入详情页）。
> - **`replace()`** 适用于重定向（如登录成功后跳转）。
> - **`go(n)`** 适用于返回或前进特定页面（如后退到上一页）。
>
> 编程式导航可以让页面跳转更灵活，比如在**事件回调、表单提交、权限校验等场景中**使用，非常实用。





### Pinia  和  VueX

> Pinia对标Vue3
>
> VueX对标Vue2
>
> 集中式状态（数据）管理——数据放在一起
>
> 共享的数据——才交给集中式状态（数据）管理



> `nanoid` 是一个**轻量级、高性能且安全的唯一 ID 生成库**，用于生成**短且随机的唯一字符串**
>
> ```cmd
> npm install nanoid
> ```



#### 引入pinia

> ```cmd
> npm install pinia
> # 或者
> yarn add pinia
> # 或者
> pnpm add pinia
> 
> ```
>
> 
>
> main.ts里注册
>
> ```main.ts
> import { createApp } from 'vue'
> import { createPinia } from 'pinia'
> import App from './App.vue'
> 
> const app = createApp(App)
> 
> // 创建 Pinia 实例并注册
> const pinia = createPinia()
> app.use(pinia)
> 
> app.mount('#app')
> 
> ```



#### 存储+读取数据

> src下新建store 放 跟组件名一样的ts文件，
>
> 比如你有一个Count.vue 就在store里建一个count.ts
>
> ```count.ts
> import {defineStore} from 'pinia'
> 
> export const useCountStore = defineStore('count',{
> 	//真正存储数据的地方
> 	state(){
> 	return{
> 		 sum:6
> 	 }
> 	}
> })
> 
> ```
>
> 
>
> 
>
> 在Count.vue里引用pinia的count.ts
>
> ```Count.vue
> <tmeplate>
> <h2>当前求和为{{countStore.sum}}</h2>
> 
> </template
> 
> <script>
> import {useCountStore} from '@/store/count' 
> 
> const countStore = useCountStore();
> 
> console.log(countStore.sum) 
> //拿到count.ts里设置的sum的值
> 
> </script>
> ```
>
> 







#### 修改数据-三种方式

> 在 `Pinia` 中，修改 `state` 的方式主要有 **三种**，分别是：
>
> 1. **直接修改 `state`**（最直接的方法）
> 2. **使用 `actions` 修改 `state`**（推荐，封装业务逻辑）
> 3. **使用 `$patch` 进行批量修改**（适合修改多个属性）
>
> ------
>
> ## **1. 直接修改 `state`**
>
> 最简单的方式，直接在组件中修改 `state` 变量。
>
> ### **示例**
>
> ```
> javascript复制编辑// stores/counter.js
> import { defineStore } from 'pinia'
> 
> export const useCounterStore = defineStore('counter', {
>   state: () => ({
>     count: 0
>   })
> })
> ```
>
> 在组件中直接修改 `state`：
>
> ```
> vue复制编辑<script setup>
> import { useCounterStore } from '@/stores/counter'
> 
> const counter = useCounterStore()
> 
> // 直接修改 state
> counter.count++  // 增加 1
> counter.count = 10  // 直接赋值
> </script>
> 
> <template>
>   <div>
>     <p>计数：{{ counter.count }}</p>
>     <button @click="counter.count++">增加</button>
>   </div>
> </template>
> ```
>
> ✅ **适用于简单的数据修改**，但不推荐用于复杂的逻辑。
>
> ------
>
> ## **2. 使用 `actions` 修改 `state`**
>
> `actions` 是 `Pinia` 提供的方法，适合封装逻辑，特别是涉及 **异步操作** 时。
>
> ### **示例**
>
> ```
> javascript复制编辑export const useCounterStore = defineStore('counter', {
>   state: () => ({
>     count: 0
>   }),
>   actions: {
>     increment() {
>       this.count++
>     },
>     async fetchCountFromAPI() {
>       const response = await fetch('https://api.example.com/count')
>       const data = await response.json()
>       this.count = data.count
>     }
>   }
> })
> ```
>
> 在组件中使用：
>
> ```
> <script setup>
> import { useCounterStore } from '@/stores/counter'
> 
> const counter = useCounterStore()
> 
> // 调用 action 修改 state
> counter.increment()
> counter.fetchCountFromAPI()
> </script>
> 
> <template>
>   <div>
>     <p>计数：{{ counter.count }}</p>
>     <button @click="counter.increment">增加</button>
>   </div>
> </template>
> ```
>
> ✅ **适用于封装业务逻辑，特别是复杂逻辑或异步操作**。
>
> ------
>
> ## **3. 使用 `$patch` 进行批量修改**
>
> 如果要**一次性修改多个 `state` 变量**，`$patch` 方式更高效。
>
> ### **示例**
>
> ```
> javascript复制编辑export const useCounterStore = defineStore('counter', {
>   state: () => ({
>     count: 0,
>     name: 'Pinia'
>   })
> })
> ```
>
> 在组件中：
>
> ```
> <script setup>
> import { useCounterStore } from '@/stores/counter'
> 
> const counter = useCounterStore()
> 
> // 使用 $patch 修改多个属性
> counter.$patch({
>   count: 100,
>   name: 'Vue Pinia'
> })
> 
> // 也可以使用回调函数修改
> counter.$patch((state) => {
>   state.count += 10
>   state.name = 'Updated Pinia'
> })
> </script>
> 
> <template>
>   <div>
>     <p>计数：{{ counter.count }}</p>
>     <p>名称：{{ counter.name }}</p>
>   </div>
> </template>
> ```
>
> ✅ **适用于批量修改多个 `state`，提高代码可读性**。
>
> ------
>
> ## **总结**
>
> | 方式                 | 适用场景            | 示例                    |
> | -------------------- | ------------------- | ----------------------- |
> | **直接修改 `state`** | 简单修改            | `counter.count++`       |
> | **使用 `actions`**   | 封装逻辑 & 处理异步 | `counter.increment()`   |
> | **使用 `$patch`**    | 批量修改多个 state  | `counter.$patch({...})` |
>
> **推荐**：对于简单场景可以直接修改 `state`，但更推荐使用 **`actions` 进行封装**，而 `$patch` 适用于**批量修改多个字段**。