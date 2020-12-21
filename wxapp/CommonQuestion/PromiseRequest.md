# setData({})遇到Promise 请求

如果接口封装了```Promise```请求，那么在页面中使用经常会遇到一个问题，在页面重新渲染时需要先重置数据，再接口调用，将返回值赋值，但小程序提示无法判定你设置的数据，其实是由于小程序的```setData({})```方法导致的，小程序因为其性能原因，可能会因为你重置数据和接口赋值都操作了同一个数据，它无法判定应该如何设置该数据而导致报错。

解决办法：
在公共方法或者接口封装页面封装睡眠方法的```Promise```请求
```javascript
// 延时
sleep(time) {
	return new Promise((resolve) => {
		let timer = setTimeout(() => {
			clearTimeout(timer)
			resolve()
		}, time)
	})
}
```
在接口响应完成后先调用等待方法，比如等待100毫秒，例
```javascript
await XXXX.sleep(100)
```
注：XXXX表示为你封装页面抛出的方法，需在调用界面先引入定义