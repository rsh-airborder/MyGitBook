<!--
 * @Author: kendrick任
 * @Date: 2020-12-18 13:02:50
 * @LastEditTime: 2020-12-21 15:01:30
 * @Description: 版本申明
 * @FilePath: \gitbook\wxapp\CommonQuestion\Share.md
 * @
-->
# 分享

这里仅叙述分享给微信好友和微信群

#### 右上角三个点分享
触发的是```Page.onShareAppMessage```事件，[官方文档](https://developers.weixin.qq.com/miniprogram/dev/reference/api/Page.html#onshareappmessageobject-object)，可配置项为```title```（转发标题），```path```（转发路径），```imageUrl```（自定义图片路径，注意：<font color="red">图片长宽比为5:4</font>）
将参数放在对象中返回即可，每个页面需单独配置，不配置则分享当前页，标题默认小程序名，图片为当前页截图
##### 参数携带
在配置```path```时，用```url```的方式方式拼接参数，例```/page/xxx?user=xxx```
##### 参数接收
两种方式
###### 1.分享页面内获取
在```onLoad```生命周期中获取，例
```javascript
onLoad: function(options) {
    // 参数在options里，可自行打印查看
}
```
###### 2.全局获取（如分享的都是同个页面，推荐）
在```app.js```的```onShow```方法内获取，需配合[场景值](https://developers.weixin.qq.com/miniprogram/dev/reference/scene-list.html)使用，例
```javascript
onShow: function (res) {
	// 判断是否为分享进入
	if(res.scene === 1007 || res.scene === 1008){
		// 分享的数据在返回值的query中
		let queryObj = res.query
		// 可存入微信小程序缓存中使用
	}
}
```

#### 按钮分享
在页面中使用```button```按钮，添加```open-type="share"```，[官方文档](https://developers.weixin.qq.com/miniprogram/dev/framework/open-ability/share.html#%E9%A1%B5%E9%9D%A2%E5%86%85%E5%8F%91%E8%B5%B7%E8%BD%AC%E5%8F%91)，点击触发的也是```Page.onShareAppMessage```事件，配置项如上，如存在该页面点击分享与右上角三个点分享有不同的业务，则可如此配置
```javascript
onShareAppMessage: function(){
	if (res.from === 'button') {
		// 来自页面内转发按钮，写按钮分享逻辑
    } else {
		// 三个点，写三个点分享逻辑
	}
}
```