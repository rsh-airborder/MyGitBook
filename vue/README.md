# vue
Vue (读音 /vjuː/，类似于 view) 是一套用于构建用户界面的渐进式框架。与其它大型框架不同的是，Vue 被设计为可以自底向上逐层应用。Vue 的核心库只关注视图层，不仅易于上手，还便于与第三方库或既有项目整合。另一方面，当与现代化的工具链以及各种支持类库结合使用时，Vue 也完全能够为复杂的单页应用提供驱动。

#### 以下章节将会通过vue2和vue3分别记录
- [vue2](vue2/README.md)
- [vue3](vue3/README.md)

#### vue2和vue3的部分区别罗列

##### 1.视图和组合式API
###### vue2
vue2中使用视图来进行页面管理，例如
```javascript
	data () {
    	return {}
    },
    computed: {},
    watch: {},
    methods: {},
    // 几个生命周期
    created () {}
    mounted () {}
```
##### vue3
vue3中仍可使用视图，但官方更推荐使用组合式API，使用```setup()```组件选项，详细可看[官网](https://v3.cn.vuejs.org/guide/composition-api-introduction.html#%E4%BB%80%E4%B9%88%E6%98%AF%E7%BB%84%E5%90%88%E5%BC%8F-api)，具体使用教程推荐[此网站](https://composition-api.vuejs.org/zh/#%E6%A6%82%E8%BF%B0)。

##### 2.typescript的支持
###### vue2
支持```typescript```，但支持度不高，坑较多/
###### vue3
全面支持```typescript```，vue3的绝大部分就是用```typescript```写的，要上手vue3需要把```typescript```早点上手了。

##### 3.数据拦截
以下转载于掘金的[时樾同学](https://juejin.cn/post/6907028003469918222)
###### vue2
采用```Object.defineProperty```
实现对象的拦截

```javascript
    let data = {
      m:234,
      n:[1,34,4,5676],
      h:{
        c:34
      }
    }
    function observer(data){
      if(typeof data === 'object'){
        Object.keys(data).forEach(key=>{
          defineReactive(data,key,data[key])
        })
      }
    }
    function defineReactive(obj,key,val){
      Object.defineProperty(obj,key,{
        get(){
          console.log('get')
          return val
        },
        set(newVal){
          console.log('set')
          if(newVal !== val ) val = newVal
        }
      })
    }
```

上面通过遍历```data```的数据，进行了一次简单的拦截；看似没有问题，但如果我们改变```data.h.c```是不会触发```set```钩子的，为什么？因为这里还没有实现递归，所以只拦截了最表面的一层，里面的则没有被拦截。

递归拦截对象

```javascript
    function defineReactive(obj,key,val){
      observer(val)
      Object.defineProperty(obj,key,{
        get(){
          console.log('get')
          return val
        },
        set(newVal){
          console.log('set')
          if(newVal !== val ) val = newVal
        }
      })
    }
```

递归拦截，只要在```defineReactive```函数再调一次```observer```函数把要拦截的值传给它就行。这样，就实现了对象的多层拦截。但是呢，现在是拦截不到数组的，当我们调用```push,pop```等方法它是不会触发```set```钩子的,为什么？因为```Object.defineProperty```压根就不支持数组的拦截。既然它不支持，那么我们只能拦截它的这些（'```push```', '```pop```', '```shift```', '```unshift```', '```splice```', '```sort```', '```reverse```'）改变自身数据的方法了。

数组的拦截

```javascript
    function arrayMethods(){
      const arrProto = Array.prototype
      const arrayMethods = Object.create(arrProto)
      const methods = ['push', 'pop', 'shift', 'unshift', 'splice', 'sort', 'reverse']
      methods.forEach(function (method) {
          const original = arrProto[method]
          Object.defineProperty(arrayMethods, method, {
              value: function v(...args) {
                  console.log('set arrayMethods')
                  return original.apply(this, args)
              }
          })
      })
      return arrayMethods
    }
```

以上就是对这些数组的原型方法进行了一个拦截，然后把它覆盖要拦截的数组的原型就行，下面简单修改一下```observer```

```javascript
    function observer(data){
      if(typeof data === 'object'){
        if(Array.isArray(data)){
          data.__proto__ = arrayMethods()
        }else{
          Object.keys(data).forEach(key=>{
            defineReactive(data,key,data[key])
          })
        }
      }
    }
```

在vue中，还会判断该```key```有没有```__proto__```,如果没有就直接把这些方法放到这个```key```的自身上，如果有就直接覆盖这个```__proto__```。

>完整代码

```javascript
    function arrayMethods(){
      const arrProto = Array.prototype
      const arrayMethods = Object.create(arrProto)
      const methods = ['push', 'pop', 'shift', 'unshift', 'splice', 'sort', 'reverse']
      methods.forEach(function (method) {
          const original = arrProto[method]
          Object.defineProperty(arrayMethods, method, {
              value: function v(...args) {
                  console.log('set arrayMethods')
                  return original.apply(this, args)
              }
          })
      })
      return arrayMethods
    }
    function observer(data){
      if(typeof data === 'object'){
        if(Array.isArray(data)){
          data.__proto__ = arrayMethods()
        }else{
          Object.keys(data).forEach(key=>{
            defineReactive(data,key,data[key])
          })
        }
      }
    }
    function defineReactive(obj,key,val){
      observer(val)
      Object.defineProperty(obj,key,{
        get(){
          console.log('get')
          return val
        },
        set(newVal){
          console.log('set')
          if(newVal !== val ) val = newVal
        }
      })
    }
    observer(data)
```
###### vue3
使用```proxy```
直接上代码
```javascript
	let data = {
      m:234,
      n:[1,34,4,5676],
      h:{
        c:34
      }
    }
    function defineReactive(obj){
      Object.keys(obj).forEach((key) => {
        if(typeof obj[key] === 'object'){
          obj[key] = defineReactive(obj[key])
        }
      })
      return new Proxy(obj,{
        get(target,key){
          console.log('get')
          return target[key]
        },
        set(target,key,val){
          console.log('set')
          return target[key] = val
        }
      })
    }
    data = defineReactive(data)
```