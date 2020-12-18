# canvas绘制

常见场景：新建内容  > 后端生成动态小程序码 > 前端显示 > 下载到手机

需求：小程序码中包含数据，虽可直接下载使用，但码一多或时间一长无法记清每个码的名称，将该码的名称与小程序合并为一张图下载至手机

这里举例小程序绘制和下载

#### 1.下载至小程序本地
```javascript
//imagePath为二维码路径，自行获取
getQrCode(imagePath){
    let _this = this;
    return new Promise((resolve,reject) =>{
        wx.downloadFile({
            url: imagePath, //二维码路径
            success: (res) => {
                if (res.statusCode === 200) {
                    console.log('下载成功')
                    resolve(res.tempFilePath)
                } else {
                    wx.showToast({
                        title: '二维码下载到本地失败',
                        icon: 'none',
                        duration: 1000,
                    })
                    reject(res)
                }
            },
            fail: (error) => {
                reject(error)
            }
        })
    })
}
```

#### 2.绘制canvas
此处举例绘制一个```400rpx * 400rpx```的图，名字在小程序上方并居中

wxml部分
```html
<view class="qr-box" id="canvas-container">
	<canvas canvas-id="myCanvas" style="width:100%;heigth:100%;" />
</view>
```

js部分
```javascript
//codeSrc为上述方法返回的本地路径，name为小程序码名
async canvasFun(codeSrc,name){
    const ctx = wx.createCanvasContext("myCanvas")
    wx.createSelectorQuery().select('#canvas-container').boundingClientRect((rect) => {
        let height = rect.height
        let width = rect.width
        ctx.setFillStyle('#ffffff')
        ctx.fillRect(0, 0, width, height)
        ctx.setFontSize(14)
        ctx.setFillStyle('#303133')
        if (codeSrc) {
            ctx.drawImage(codeSrc, 15, 30, width - 30, height - 30)
        }
        if (name) {
            const metrics = ctx.measureText(name).width //测量文本信息
            ctx.fillText(name, (width / 2) - (metrics / 2), 20)
            ctx.stroke()
            ctx.fill()
        }
    }).exec()
    setTimeout(() => {
        ctx.draw()
    },500)
}
```

wxss部分
```css
.qr-box{
	width: 400rpx;
	height: 400rpx;
	margin: 0 auto;
}
```

#### 下载到手机
页面制作下载按钮，点击下载

###### 下载方法
```javascript
// 保存图片
downloadPic(){
    let _this = this
    // 先获取保存到相册权限
    wx.getSetting({
        success(authTemp) {
            if (authTemp.authSetting['scope.writePhotosAlbum'] != undefined && authTemp.authSetting['scope.writePhotosAlbum'] == false) {
                // 打开自定义授权弹窗，showAuth控制弹窗显示隐藏
                _this.setData({
                    showAuth: true
                })
            }else{
                // 获取权限成功，canvas转图片
                wx.canvasToTempFilePath({
                    canvasId: 'myCanvas',
                    success: function (res) {
                        let tempFilePath = res.tempFilePath
                        wx.saveImageToPhotosAlbum({
                            filePath: tempFilePath,
                            success() {
                                wx.showToast({
                                    title: '图片保存成功',
                                    icon: 'none',
                                    duration: 1000
                                })
                            },
                            fail: function () {
                                wx.showToast({
                                    title: '图片保存失败',
                                    icon: 'none',
                                    duration: 1000
                                })
                            }
                        })
                    }
                })
            }
        }
    })
}
```

###### 自定义相册授权弹窗

wxml文件
```html
<block wx:if="{{ showAuth }}">
    <view class="join-box">
        <view class="join-content-box">
            <view class="join-content-title">提示</view>
            <view class="join-content">允许保存图片到你的相册？</view>
        </view>
        <view class="join-button-box">
            <button class="join-button-box-left" bindtap="refuseAuth">取消</button>
            <button class="join-button-box-right" bindopensetting="confirmAuth" hover-class="none" open-type='openSetting'>确定</button>
        </view>
    </view>
</block>
```
注：弹窗背景遮罩自行制作

js文件
```javascript
// 自定义授权取消
refuseAuth(){
    this.setData({
        showAuth: false
    })
},
// 自定义授权确认
confirmAuth(res){
    if (res.detail.authSetting['scope.writePhotosAlbum'] == true) {
    	this.downloadPic()
    } else {
        wx.showToast({
        	title: '保存到相册授权未打开！',
        	icon: 'none'
        })
    }
    this.setData({
    	showAuth: false
    })
},
```

wxss文件
```css
.join-box{
    position: fixed;
    z-index: 11;
    width: 80%;
    height: 276rpx;
    left: 10%;
    top: 35%;
    background-color: #fff;
    border-radius: 8rpx;
    border: 1px solid #eee;
}
.join-content-box{
    height: 180rpx;
    padding: 40rpx 48rpx 0;
}
.join-content-title{
    text-align: center;
    font-size: 28rpx;
    font-family: PingFangSC-Medium, PingFang SC;
    font-weight: 600;
    color: #303133;
    line-height: 40rpx;
    margin-bottom: 20rpx;
}
.join-content{
    font-size: 28rpx;
    font-family: PingFangSC-Regular, PingFang SC;
    font-weight: 400;
    color: #606266;
    line-height: 40rpx;
    text-align: center;
}
.join-max-box{
    position: fixed;
    z-index: 11;
    width: 80%;
    height: 396rpx;
    left: 10%;
    top: 30%;
    background-color: #fff;
    border-radius: 8rpx;
}
.join-max-content-box{
    height: 300rpx;
    padding: 40rpx 48rpx 0;
}
.join-button-box{
    height: calc(96rpx - 2px);
    display: flex;
    align-items: center;
    justify-content: center;
    border-top: 1px solid #EBEDF0;
}
.join-button-box-left{
    width: 50%;
    color: #303133;
    height: 100%;
    display: flex;
    align-items: center;
    justify-content: center;
    background-color: #fff;
    border: none;
    border-right: 1px solid #EBEDF0;
    border-radius: unset;
    padding: 0;
    font-size: 28rpx;
    font-family: PingFangSC-Regular, PingFang SC;
    font-weight: 400;
    line-height: 40rpx;
}
.join-button-box-left::after{
    border: none;
}
.join-button-box-right{
    width: 50%;
    height: 100%;
    display: flex;
    align-items: center;
    justify-content: center;
    background-color: #fff;
    border: none;
    border-radius: unset;
    padding: 0;
    font-size: 28rpx;
    font-family: PingFangSC-Regular, PingFang SC;
    font-weight: 400;
    color: #036ED5;
    line-height: 40rpx;
}
.join-button-box-right::after{
    border: none;
}
```