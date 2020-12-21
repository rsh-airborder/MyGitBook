<!--
 * @Author: kendrick任
 * @Date: 2020-12-17 11:06:32
 * @LastEditTime: 2020-12-21 15:01:34
 * @Description: 版本申明
 * @FilePath: \gitbook\wxapp\CommonQuestion\UseEcharts.md
 * @
-->
# 微信小程序中Echarts的使用

#### 使用
使用我这里不多加赘述，详细请看[echarts-for-weixin](https://github.com/ecomfe/echarts-for-weixin)。

#### 无法滑动与层级过高的问题

##### 无法滑动
因为微信小程序上的echarts是用canvas画的，所以很多问题的产生是canvas导致的，在微信小程序中，canvas是作为[原生组件](https://developers.weixin.qq.com/miniprogram/dev/component/native-component.html#%E5%8E%9F%E7%94%9F%E7%BB%84%E4%BB%B6%E7%9A%84%E4%BD%BF%E7%94%A8%E9%99%90%E5%88%B6)存在，有着许多的限制。
在项目中，如果发现存在echarts的页面无法滑动，在你的echarts绘制wxml上加上属性```disable-scroll="true"```即可，例如：
```html
{% raw %}
<view class="echarts-box">
	<ec-canvas id="mychart-dom-bar" canvas-id="mychart-bar" ec="{{ ec }}" disable-scroll="true"></ec-canvas>
</view>
{% endraw %}
```
注：包裹echarts的组件盒子必须为最高级，外部不可以在添加任何的块，也不能设置任何的position属性。

##### 层级过高
层级过高也是canvas原生组件导致的，要覆盖在canvas上层，仅可使用cover-view,cover-image，[小程序的官方文档](https://developers.weixin.qq.com/miniprogram/dev/component/native-component.html#%E5%8E%9F%E7%94%9F%E7%BB%84%E4%BB%B6%E7%9A%84%E4%BD%BF%E7%94%A8%E9%99%90%E5%88%B6)有说明。

#### 小程序中echarts.js文件过大
echarts官方提供了一个[在线定制网站](https://echarts.apache.org/zh/builder.html)，勾选你项目需要的配置项，构建出的echarts.min.js重命名为echarts.js，替换你项目中的echarts.js即可。

#### 限制
不可自定义```tooltip```，小程序中```tooltip```的```formatter```方法不可返回标签字符串，仅可使用普通字符串文本。