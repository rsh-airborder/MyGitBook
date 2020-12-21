<!--
 * @Author: kendrick任
 * @Date: 2020-12-17 11:06:32
 * @LastEditTime: 2020-12-21 16:24:19
 * @Description: 版本申明
 * @FilePath: \gitbook\wxapp\PublicFunction.md
 * @
-->
# 公共方法封装

### 在utils文件夹下的util.js文件存放方法

##### 防抖
```javascript
function debounce(fn, wait) {
	let timer = null
	return () => {
		if (timer) {
			clearTimeout(timer)
		}
		timer = setTimeout(fn, wait)
	}
}
```
##### 节流
```javascript
function throttle(fn, delay){
	let valid = true
	return () => {
		if(!valid){
			//休息时间 暂不接客
			return false 
		}
		// 工作时间，执行函数并且在间隔期内把状态位设为无效
		valid = false
		setTimeout(() => {
			fn()
			valid = true
		}, delay)
	}
}
```
##### 时间戳转时间（全部带补零）
```javascript
function timeToString(time,num){
	let new_date = new Date(time)
	// YYYY/MM/DD HH:mm:ss 格式（年/月/日 时:分:秒）
	if(num === 1){
		new_date = new_date.getFullYear().toString() + '/' + ((new_date.getMonth() + 1) < 10 ? '0' + (new_date.getMonth() + 1) : (new_date.getMonth() + 1)) + '/' + (new_date.getDate() < 10 ? '0' + new_date.getDate() : new_date.getDate()) + ' ' + (new_date.getHours() < 10 ? '0' + new_date.getHours() : new_date.getHours()) + ':' + (new_date.getMinutes() < 10 ? "0" + new_date.getMinutes() : new_date.getMinutes())
	}
	// YY/MM/DD HH:mm:ss 格式（年/月/日 时:分:秒）年只截取后两位
	else if(num === 2){
		new_date = new_date.getFullYear().toString().substr(2) + '/' + ((new_date.getMonth() + 1) < 10 ? '0' + (new_date.getMonth() + 1) : (new_date.getMonth() + 1)) + '/' + (new_date.getDate() < 10 ? '0' + new_date.getDate() : new_date.getDate()) + ' ' + (new_date.getHours() < 10 ? '0' + new_date.getHours() : new_date.getHours()) + ':' + (new_date.getMinutes() < 10 ? "0" + new_date.getMinutes() : new_date.getMinutes())
	}
	// MM/DD HH:mm 格式（月/日 时:分）
	else if(num === 3){
		new_date = ((new_date.getMonth() + 1) < 10 ? '0' + (new_date.getMonth() + 1) : (new_date.getMonth() + 1)) + '/' + (new_date.getDate() < 10 ? '0' + new_date.getDate() : new_date.getDate()) + ' ' + (new_date.getHours() < 10 ? '0' + new_date.getHours() : new_date.getHours()) + ':' + (new_date.getMinutes() < 10 ? "0" + new_date.getMinutes() : new_date.getMinutes())
	}
	return new_date
}
```
##### base64编码，配合encodeURIComponent使用
```javascript
function base64_encode (str) {
	var c1, c2, c3;
	var base64EncodeChars = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/=";
	var i = 0, len = str.length, strin = '';
	while (i < len) {
		c1 = str.charCodeAt(i++) & 0xff;
		if (i == len) {
			strin += base64EncodeChars.charAt(c1 >> 2);
			strin += base64EncodeChars.charAt((c1 & 0x3) << 4);
			strin += "==";
			break;
		}
		c2 = str.charCodeAt(i++);
		if (i == len) {
			strin += base64EncodeChars.charAt(c1 >> 2);
			strin += base64EncodeChars.charAt(((c1 & 0x3) << 4) | ((c2 & 0xF0) >> 4));
			strin += base64EncodeChars.charAt((c2 & 0xF) << 2);
			strin += "=";
			break;
		}
		c3 = str.charCodeAt(i++);
		strin += base64EncodeChars.charAt(c1 >> 2);
		strin += base64EncodeChars.charAt(((c1 & 0x3) << 4) | ((c2 & 0xF0) >> 4));
		strin += base64EncodeChars.charAt(((c2 & 0xF) << 2) | ((c3 & 0xC0) >> 6));
		strin += base64EncodeChars.charAt(c3 & 0x3F)
	}
	return strin
}
```
##### base64解码，配合decodeURIComponent使用
```javascript
function base64_decode (input) {
	var base64EncodeChars = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/=";
	var output = "";
	var chr1, chr2, chr3;
	var enc1, enc2, enc3, enc4;
	var i = 0;
	input = input.replace(/[^A-Za-z0-9\+\/\=]/g, "");
	while (i < input.length) {
		enc1 = base64EncodeChars.indexOf(input.charAt(i++));
		enc2 = base64EncodeChars.indexOf(input.charAt(i++));
		enc3 = base64EncodeChars.indexOf(input.charAt(i++));
		enc4 = base64EncodeChars.indexOf(input.charAt(i++));
		chr1 = (enc1 << 2) | (enc2 >> 4);
		chr2 = ((enc2 & 15) << 4) | (enc3 >> 2);
		chr3 = ((enc3 & 3) << 6) | enc4;
		output = output + String.fromCharCode(chr1);
		if (enc3 != 64) {
			output = output + String.fromCharCode(chr2);
		}
		if (enc4 != 64) {
			output = output + String.fromCharCode(chr3);
		}
	}
	return utf8_decode(output);
}
```
##### utf-8解码
```javascript
function utf8_decode (utftext) {
	var string = '';
	let i = 0;
	let c = 0;
	let c1 = 0;
	let c2 = 0;
	while (i < utftext.length) {
		c = utftext.charCodeAt(i);
		if (c < 128) {
			string += String.fromCharCode(c);
			i++;
		} else if ((c > 191) && (c < 224)) {
			c1 = utftext.charCodeAt(i + 1);
			string += String.fromCharCode(((c & 31) << 6) | (c1 & 63));
			i += 2;
		} else {
			c1 = utftext.charCodeAt(i + 1);
			c2 = utftext.charCodeAt(i + 2);
			string += String.fromCharCode(((c & 15) << 12) | ((c1 & 63) << 6) | (c2 & 63));
			i += 3;
		}
	}
	return string;
}
```
##### 方法抛出
```
module.exports = {
	debounce: debounce,
	throttle: throttle,
	timeToString: timeToString,
	base64_encode: base64_encode,
	base64_decode: base64_decode,
	utf8_decode: utf8_decode
}
```
##### 页面中使用
```javascript
// 引入
let utils = require('相对路径/utils/util.js')
// 使用
utils.方法名
```