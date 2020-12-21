<!--
 * @Author: kendrick任
 * @Date: 2020-12-17 11:06:32
 * @LastEditTime: 2020-12-21 15:00:43
 * @Description: 版本申明
 * @FilePath: \gitbook\wxapp\RequestPackage.md
 * @
-->
# 请求封装

### 在utils文件夹下建立base.js文件
```javascript
let UrlConfig = {}
UrlConfig.url = '此处填写公共url'

class Base {
	// 图片上传（此处举例的是上传阿里云oss,如正常传服务器上，可以类比request封装）
	async uploadOss(options){
		let ossData = await this.ossRequest()
		ossData = ossData.data
		return new Promise((resolve, reject) =>{
			wx.uploadFile({
				filePath: options.file,
				name: 'file',
				url: ossData.requestUri,
				// formData里的具体参数根据你获取到的ossSign数据来传
				formData: {
					'x-oss-meta-tag': 'dummy_etag_xxx', //我们公司阿里云设置要求
					'key': ossData.bucketNameFile + '$' + '{filename}', // 阿里云AccesKey ID
					'policy': ossData.encPolicy, // 加密规则
					'OSSAccessKeyId': ossData.accessKeyId, // 阿里云 Access Key Secret
					'signature': ossData.signature,  // 签名
					'success_action_status': '200'  // 固定值200
				},
				success(res) {
					if (res.statusCode == 200) {
						console.log('图片上传成功200')
						let filename = options.file
						filename = filename.split('//')[1]
						if(filename.indexOf('/') > -1){
							filename = filename.substring(4)
						}
						resolve(`${ossData.url}${filename}`)
					} else {
						console.log('图片上传成功不是200')
						reject(res)
					}
				},
				fail (err) {
					console.log('图片上传失败',err)
					reject(err)
				}
			})
		})
	}
	// 请求封装
	async request(param) {
		if (!param.method) {
			param.method = 'GET'
		}
		var nowTime = parseInt(new Date().getTime() / 1000)
		var expired = Number(wx.getStorageSync('expired'))
		var token
		if(nowTime > expired || !wx.getStorageSync('token')){
			let tokenData = await this.login()
			wx.setStorageSync('token',tokenData.token_type + ' ' + tokenData.access_token)
			wx.setStorageSync('expired',tokenData.profile.expires_at)
			token = tokenData.token_type + ' ' + tokenData.access_token
		}else{
			token = wx.getStorageSync('token')
		}
		return new Promise((resolve, reject) =>{
			wx.request({
				url: UrlConfig.url + param.url,
				method: param.method,
				dataType: 'json',
				header: {
					'Content-Type': 'application/json',
					'authorization': token
				},
				data: param.data,
				success(res) {
					if (res.data.code == 100) {
						resolve(res.data)
					} else {
						//错误信息可根据code值再次封装
						reject(res)
					}
				},
				fail (err) {
					wx.showToast({
						title: '网络错误',
						icon: 'none'
					})
					reject(err)
				}
			})
		})
	}
	// 获取ossSign
	async ossRequest() {
		var nowTime = parseInt(new Date().getTime() / 1000)
		var expired = Number(wx.getStorageSync('expired'))
		var token
		if(nowTime > expired || !wx.getStorageSync('token')){
			let tokenData = await this.login()
			wx.setStorageSync('token',tokenData.token_type + ' ' + tokenData.access_token)
			wx.setStorageSync('expired',tokenData.profile.expires_at)
			token = tokenData.token_type + ' ' + tokenData.access_token
		}else{
			token = wx.getStorageSync('token')
		}
		return new Promise((resolve, reject) =>{
			wx.request({
				url: UrlConfig.url + '获取ossSign的接口',
				method: 'GET',
				dataType: 'json',
				header: {
					'Content-Type': 'application/json',
					'authorization': token
				},
				success(res) {
					resolve(res)
				},
				fail (err) {
					reject(err)
				}
			})
		})
	}
	// 登录
	login() {
		let that = this
		return new Promise((resolve, reject) =>{
			wx.login({
				success: (res) => {
					if (res.code) {
						// 发起获取令牌请求
						// console.log(res.code)
						wx.request({
							method: 'POST',
							url: UrlConfig.url + 'Auth/AuthorizeWXSDK',
							data: {
								// appType值根据不同项目变化
								appType: 1001,
								js_code: res.code
							},
							dataType: 'json',
							success: (e) => {
								resolve(e.data)
							}
						})
					} else {
						that.login()
					}
				},
				fail (err) {
					reject(err)
				}
			})
		})
	}
	// 延时
	sleep(time) {
		return new Promise((resolve) => {
			let timer = setTimeout(() => {
				clearTimeout(timer)
				resolve()
			}, time)
		})
	}
}

export { Base }
```
### 在app.js中使用
```javascript
// 引入
import { Base } from 'utils/base.js'
var base = new Base()

App({
	onLaunch: async function () {
		if (wx.getStorageSync('token') === '') {
			console.log('token失效')
			let tokenData = await base.login()
			wx.setStorageSync('token',tokenData.token_type + ' ' + tokenData.access_token)
			wx.setStorageSync('expired',tokenData.profile.expires_at)
		} else {
			console.log('token获取成功')
		}
	},
	globalData: {
		userInfo: null
	}
})
```
### 其他页面中使用
```javascript
// 引入
import { Base } from '相对路径/utils/base.js'
var base = new Base()
// 使用
base.request({
	···
})
base.upload({
	···
})
```