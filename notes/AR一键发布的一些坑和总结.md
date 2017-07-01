# AR一键发布的一些坑和总结


* ## 二维码相关
    项目要求在页面中插入二维码，二维码的具体内容是网页的网址
> 我用vue写的网页，二维码生成使用了第三方插件qrcode, 版本信息是"qrcode": "^0.8.2"
>
> 二维码生成的原理是利用canvas来绘制每一个像素点，所以在用插件的时候需要在html中先创建一个canvas容器。下面是我的代码。


```html
<!--html-->
<template>
    <div id='code'></div>
	<canvas id="canvas"></canvas>
    </div>
</template>

```


```js
//安装
npm install --save-dev qrcode

//调用
<script>
    import QRCode from 'qrcode'
    Vue.use(QRCode)
    
    export default{
		methods:{
			useqrcode(){
				var url = "http://www.baidu.com"
				var canvas = document.getElementById('canvas')
				QRCode.toCanvas(canvas, url, function (error) {
				    if (error) console.error(error)
				})
				//到上面这步为止二维码已经画在了canvas容器里，如果不用转成图像的话到这步就行了
				
				//如果想把canvas转成图像，需要调用canvas的toDataURL方法
				//canvas.toDataURL(type, encoderOptions);
				//type:图片类型  encoderOptions：图片质量，常在图片压缩时使用
				var type="image/png";
    			        this.dataurl = document.getElementById('canvas').toDataURL('image/png');
			}
		},
		mounted(){
			this.useqrcode();
		}
	}
</script>

```
* ## 浏览器类型判断

> 因为要兼容移动端，而且本App只适用于IOS版本，所以免不了IOS和Androd的检查；且App只能在浏览器中下载，还需要检查当前环境如果是微信或者QQ或者微博的话需要给出提示信息。免不了一堆的版本检测。下面贴一下主流版本的检测方法：

```js
var browser = {
        versions: function() {
            var u = navigator.userAgent;
            return { //移动终端浏览器版本信息
                mobile: !!u.match(/AppleWebKit.*Mobile.*/), //是否为移动终端
                ios: !!u.match(/\(i[^;]+;( U;)? CPU.+Mac OS X/), //ios终端
                // android: u.indexOf('Android') > -1 || u.indexOf('Linux') > -1, //android终端或uc浏览器
                iPhone: u.indexOf('iPhone') > -1, //是否为iPhone或者QQHD浏览器
                iPad: u.indexOf('iPad') > -1, //是否iPad
                Android:u.indexOf('Android') > -1 ,//是否为安卓
                Winxin: u.toLowerCase().match(/MicroMessenger/i)=='micromessenger',//是否为微信
                isQQ : u.toLowerCase().match(/QQ/i) == "qq",//是否为QQ
                safari : u.indexOf('Safari') > -1 //是否是Safari
            };
        }()
}

```
* ## loading加载圈的实现
> 利用CSS3实现一个圆一个在转的动态loading效果
>
> [放一个阮一峰的CSS动画讲解地址](http://www.ruanyifeng.com/blog/2014/02/css_transition_and_animation.html)

```html
<div class="loading "></div>
```
```css
.loading{
        background-color: transparent;
        width: 45px;//宽45
        height: 45px;//高45
        border: 1px solid #1abc9c;//边框颜色
        border-radius: 50%;//圆角，正方形变成一个圆
        border-top-color: transparent;
        border-left-color: transparent;//制造效果，让圈只有一半
        -webkit-animation: rotate .6s linear infinite;
        animation: rotate .6s linear infinite;//动画，旋转 0.6s完成 无限次
        margin:auto;
    }
    
    //从旋转0°到360°
    @keyframes rotate{
      0% {
        -webkit-transform: rotate(0);
        transform: rotate(0);
      }
      100% {
         -webkit-transform: rotate(360deg);
        transform: rotate(360deg);
      }
    }
```

* ## 关于无痕浏览
>说这个之前首先给大家推荐一个前端工具     [http://console.hongliang.org/](http://console.hongliang.org/)，打开后直接选择guest浏览就可以
> 
> 在项目里面加上这个工具页面的script引用，然后用手机打开你项目的时候的console调试信息就会输出在下方的log框中，这样再也不愁找不到移动端的错误了。
> 
>言归正传，这个页面做完，IOS和Android我都分别在微信QQ还有普通浏览器里面测试了一下，哦都很完美。BUT！！！在展示的时候出问题了，我看了一下输出的报错信息。
```js
QuotaExceededError (DOM Exception 22): The quota has been exceeded.
```
> 百度了一下之后明白了，是用户开启了无痕浏览模式。我在项目里面用到了localstorage来存储信息，当用户开启无痕浏览的时候，通过window.localStorage来检查浏览器是否支持storage的时候还是会显示支持，只是在你set的时候会直接报错。我就是在这边报的错。解决方法就是把storage的相关方法放到try和catch中，捕获异常。

```js
if(window.localStorage){
	try{
		var storage=window.localStorage;
		storage.setItem('qrUrl',qrUrl);
	    }catch(e){//应该是无痕模式
		global.qrUrl = qrUrl;
	    }
}
```
> 大概就是以上这些，另外还有一些webpack的使用和配置问题，包括利用proxy实现用户代理，还有打包的时候坑，包括一些静态的图片总是404 not found 。这些问题我现在还是只知道皮毛，周末系统看下webpack之后再统一总结下吧。
>
> 另：markdown真好用(●′ω`●) 