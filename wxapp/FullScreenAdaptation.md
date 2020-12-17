# IOS全面屏的底栏适配

背景：ios从iphone X开始进入全面屏时代，底栏出现了代替home键的横条，横条所在区域被称之为安全区域，这里指的全面屏适配就是适配安全区域，防止误触。

### 第一种：使用wx.getSystemInfo()
在app.js中使用
```javascript
onLaunch: function (res) {
    wx.getSystemInfo({
        success(res){
            // 判断是不是iphone X及以上机型
            if (res.model.search('iPhone X') != -1){
                // 保存全局变量
                app.globalData.isIpx = true
            }else{
                app.globalData.isIpx = false
            }
            // 获取安全区域右下角纵坐标
            let height1 = res.safeArea.bottom
            // 获取屏幕高度
            let height2 = res.screenHeight
            // 相减为安全区域高度
            let subHeight = height2 - height1
            // 存缓存里
            wx.setStorageSync('subHeight',subHeight)
        }
    })
}
```
### 第二种：使用css
[微信小程序官方](https://developers.weixin.qq.com/miniprogram/dev/framework/audits/accessibility.html#5.%20iPhone%20X%20%E5%85%BC%E5%AE%B9)的兼容提案
```css
padding-bottom: constant(safe-area-inset-bottom);
padding-bottom: env(safe-area-inset-bottom);
```
如果遇到当前页面还有```scroll-view```标签，要动态计算高度时可配合```wx.getSystemInfo()```使用