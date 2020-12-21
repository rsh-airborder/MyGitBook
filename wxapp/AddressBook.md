<!--
 * @Author: kendrick任
 * @Date: 2020-12-18 13:54:49
 * @LastEditTime: 2020-12-21 15:01:02
 * @Description: 版本申明
 * @FilePath: \gitbook\wxapp\AddressBook.md
 * @
-->
# 通讯录制作

可使用```vant```的```IndexBar```组件，[官方文档](https://vant-contrib.gitee.io/vant-weapp/#/index-bar)，但因为这个组件外部不可在套用块（会导致右侧索引失效）以及全屏仅有此组件时才能完美触发所有方法导致业务操作困难，故自定义一通讯录。

wxml文件
```html
<!-- 主滚动页 -->
{% raw %}
<scroll-view 
    scroll-y="{{ true }}" 
    class="scroll-anchor" 
    scroll-top="{{ scrollTop }}" 
    scroll-with-animation="{{ true }}" 
    bindscroll="scrollFun"
    enable-back-to-top="{{ true }}"
>
    <block wx:for="{{ peopleList }}" wx:key="index">
    	<!-- 锚点 -->
        <view 
            class="{{ activeIndex === index ? 'anchor-title-box activeIndex-anchor-title-box' : 'anchor-title-box'}}"
        >
        	{{ item.anchor }}
        </view>
        <!-- 人员 -->
        <block wx:for="{{ item.list }}" wx:for-index="i" wx:for-item="v" wx:key="i">
            <view class="people-list">
            	<!-- 头像 -->
                <image src="{{ v.avatarUrl }}" class="avatar" lazy-load />
                <!-- 昵称 -->
                <rich-text class="people-list-name" nodes="{{ v.nickName }}"></rich-text>
                <!-- 是否为新成员 -->
                <image src="新成员图片链接" class="people-new-image" wx:if="{{ v.isNew }}" />
        	</view>
        </block>
    </block>
</scroll-view>
<!-- 侧栏索引 -->
<view class="anchor-box">
    <block wx:for="{{ indexList }}" wx:key="index">
        <view 
        	class="{{ activeIndex === index ? 'anchor-text active-anchor-text' : 'anchor-text'}}" 
            bindtap="anchorFun" 
            data-index="{{ index }}"
        >
        {{ item }}
        </view>
    </block>
</view>
{% endraw %}
```
注：```scroll-view```的配置请看[官方文档](https://developers.weixin.qq.com/miniprogram/dev/component/scroll-view.html)

js文件
```javascript
data: {
    // 人员列表
    peopleList: [],
    // 索引列表
    indexList: [],
    // 默认激活下标
    activeIndex: 0,
    // 设置滚动高度
    scrollTop: 0,
    // 标识是否为点击滚动，true为点击索引触发的滚动，false则不是
    bindScroll: false,
}
// 右侧跳转
anchorFun(e){
    let index = e.currentTarget.dataset.index
    let winWith = wx.getSystemInfoSync().windowWidth
    let topHeight = 0
    let peopleList = this.data.peopleList
    for(let i = 0; i < index; i++){
        topHeight = topHeight + 30 / 375 * winWith
        topHeight = topHeight + ( 50 / 375 * winWith ) * peopleList[i].list.length
    }
    this.setData({
        activeIndex: index,
        scrollTop: topHeight,
        bindScroll: true
    })
},
// 滚动监听
scrollFun(e){
    let _this = this
    let scrollTop = e.detail.scrollTop
    if(_this.data.bindScroll){
        _this.setData({
       		bindScroll: false
        })
    }else{
        let topList = _this.data.topList
        for(let i = 1; i < topList.length; i++){
            if(scrollTop <= topList[i] && scrollTop >= topList[i-1]){
                _this.setData({
                	activeIndex: i - 1
                })
            }
        }
    }
},
// 人员排序方法
peopleListSortFun(peopleList){
    let list = [
        {
            anchor: 'A',
            list: []
        },
        {
            anchor: 'B',
            list: []
        },
        {
            anchor: 'C',
            list: []
        },
        {
            anchor: 'D',
            list: []
        },
        {
            anchor: 'E',
            list: []
        },
        {
            anchor: 'F',
            list: []
        },
        {
            anchor: 'G',
            list: []
        },
        {
            anchor: 'H',
            list: []
        },
        {
            anchor: 'I',
            list: []
        },
        {
            anchor: 'J',
            list: []
        },
        {
            anchor: 'K',
            list: []
        },
        {
            anchor: 'L',
            list: []
        },
        {
            anchor: 'M',
            list: []
        },
        {
            anchor: 'N',
            list: []
        },
        {
            anchor: 'O',
            list: []
        },
        {
            anchor: 'P',
            list: []
        },
        {
            anchor: 'Q',
            list: []
        },
        {
            anchor: 'R',
            list: []
        },
        {
            anchor: 'S',
            list: []
        },
        {
            anchor: 'T',
            list: []
        },
        {
            anchor: 'U',
            list: []
        },
        {
            anchor: 'V',
            list: []
        },
        {
            anchor: 'W',
            list: []
        },
        {
            anchor: 'X',
            list: []
        },
        {
            anchor: 'Y',
            list: []
        },
        {
            anchor: 'Z',
            list: []
        },
        {
            anchor: '#',
            list: []
        }
    ]
    let charList = ['A','B','C','D','E','F','G','H','I','J','K','L','M','N','O','P','Q','R','S','T','U','V','W','X','Y','Z']
    peopleList.forEach(item => {
    	// 只取名称的第一个字符
        let firstStr = item.nickName.substr(0,1)
        // 使用pinyin插件将其转为拼音，仅包含拼音的第一个字母
        let firstPY = pinyin.getFirstLetter(firstStr)
        // 转大写
        firstPY = firstPY.toLocaleUpperCase()
        // 将不满足条件的重设为#
        if(charList.indexOf(firstPY) === -1){
        	firstPY = '#'
    	}
    	// 放入数组
        list.forEach(value => {
            if(value.anchor === firstPY){
            	value.list.push(item)
            }
        })
    })
    // 将所有空数组过滤
    let lastList = list.filter(item => {
        if(item.list.length > 0){
        	return item
        }
    })
    return lastList
},
// 初始人员列表设置
peopleFun(peopleList){
    peopleList = this.peopleListSortFun(peopleList)
    let num = 0
    let allSearchList = []
    let indexList = []
    let topList = [0]
    let topHeight = 0
    let winWith = wx.getSystemInfoSync().windowWidth
    // 记录每个锚点需滚动多少
    for(let i = 0; i < peopleList.length - 1; i++){
        topHeight = topHeight + 30 / 375 * winWith
        topHeight = topHeight + ( 50 / 375 * winWith ) * peopleList[i].list.length
        topList.push(topHeight)
    }
    peopleList.forEach(item => {
        indexList.push(item.anchor)
    })
    this.setData({
        peopleList: peopleList,
        indexList: indexList,
        topList: topList
    })
},
// 调用
onShow: function() {
	this.peopleFun(接口获取到的数据)
}
```
注：文字转拼音插件的使用，这里我用的是```wl-pinyin```，[使用方法](https://cloud.tencent.com/developer/article/1605349)

wxss文件
```css
/* 主滚动页 */
.scroll-anchor{
	/* 高度根据实际项目变化 */
	height: 100vh;
}
.anchor-title-box{
    height: 60rpx;
    padding: 0 32rpx;
    display: flex;
    align-items: center;
    font-size: 28rpx;
    color: #303133;
}
.activeIndex-anchor-title-box{
    color: #036ED5;
}
.people-list{
    height: 100rpx;
    width: 100%;
    display: flex;
    align-items: center;
    border-top: 1px solid #F0F1F3;
    background-color: #fff;
    padding: 0 32rpx;
}
.avatar{
    height: 68rpx;
    width: 68rpx;
    margin-right: 40rpx;
    border-radius: 50%;
    box-shadow: 0 0 3px 0px rgba(0, 0, 0, 0.35);
}
.people-list-name{
    font-size: 28rpx;
    font-family: PingFangSC-Regular, PingFang SC;
    font-weight: 400;
    color: #303133;
    line-height: 40rpx;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
    max-width: calc(100% - 168rpx);
}
.people-new-image{
    height: 40rpx;
    width: 40rpx;
    margin-left: 20rpx;
}
.people-list:last-of-type{
    border-bottom: 1px solid #F0F1F3;
}
/* 侧栏索引 */
.anchor-box{
    position: fixed;
    width: 32rpx;
    height: 100%;
    top: 0;
    right: 6rpx;
    background-color: transparent;
    display: flex;
    align-items: center;
    flex-direction: column;
    justify-content: center;
    z-index: 9;
}
.anchor-box .anchor-text{
    height: 32rpx;
    display: flex;
    align-items: center;
    justify-content: center;
    width: 100%;
    font-size: 24rpx;
    color: #C0C4CC;
    margin-bottom: 10rpx;
}
.anchor-box .active-anchor-text{
    background-color: #606266;
    border-radius: 50%;
}
```