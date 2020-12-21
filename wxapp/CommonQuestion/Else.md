# 其他注意事项

#### 1.伪类及选择器
小程序仅支持的伪类仅有```before```和```after```两种，选择器也仅支持类选择器，id选择器，标签选择器，兄弟选择器，详情请看[官方文档](https://developers.weixin.qq.com/miniprogram/dev/framework/view/wxss.html)。

#### 2.图片预览触发onShow事件
小程序中调用图片预览一般使用小程序的官方api，```wx.previewImage```方法，使用方法请看[官方文档](https://developers.weixin.qq.com/miniprogram/dev/api/media/image/wx.previewImage.html)，而在使用过程中你会发现图片预览结束会触发该页面的```onShow```方法，但往往很多重新渲染逻辑都写在此方法中，但仅预览图片并不想触发重新渲染，解决办法，例
```javascript
//在data里定义一个是否预览的参数
data: {
	// 没在预览
    isNotPreview: false,
},
//在预览方法中重定义，方法名可自定
imagePreview(e) {
    this.setData({
    	isNotPreview: true
    })
},
//在onShow方法最上定义，判断现在是预览结束后触发的刷新，重置预览参数，直接退出后续方法
onShow: function() {
    if(this.data.isNotPreview){
        this.setData({
        	isNotPreview: false
        })
        return
    }
}
```

#### 3.分享完触发onShow事件
无论是三个点分享还是按钮点击分享，在分享完自动返回页面后也如预览一样会触发该页面的```onShow```方法，解决方法也类似。
```javascript
//在data里定义一个是否分享的参数
data: {
	// 没在分享
    isPageShare: false,
},
//在onShow方法最上定义，判断现在是分享结束后触发的刷新，重置分享参数，直接退出后续方法
onShow: function() {
    if(this.data.isPageShare){
        this.setData({
        	isPageShare: false
        })
        return
    }
},
//在分享方法中重定义，方法名固定
onShareAppMessage: function() {
    this.setData({
    	isPageShare: true
    })
}
```

#### 4.小程序未关闭即锁屏或直接切换应用或直接回主页后再次打开微信直接进小程序处理
此处的处理与你在```app.js```里处理了```scene```[场景值](https://developers.weixin.qq.com/miniprogram/dev/reference/scene-list.html)有关，因为小程序从哪种场景值进入，在不退出小程序的情况下再次触发小程序重载（例如标题描述的几种情况皆会触发），其场景值仍旧不变，如果逻辑仅判断场景值则会导致某些错误，解决办法，往往需判断场景值的情况下都是要接收该场景值时所传递的参数，仅需<font color="red">判断是否包含所需参数</font>即可，因为上述情况重载仅保留了场景值，参数皆为空值。