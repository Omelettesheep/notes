# C-T项目的一些坑和总结

> 啊，这是我实习做的第二个项目吧。介绍下这个C-T项目的话就是一个图像识别的项目，然后需要我做一个前端的demo。简单来说。。我要做的工作就是。。。上传图片！半天就写完了这个东东。。。然而没想到后面会有呢么多坑。

> * ### 样式
> 
> h5的图片上传控件是酱婶儿的
```html
<input type="file" />
```
>![image](http://note.youdao.com/yws/api/personal/file/5AADE603D79549F28FB5A2F0CC4CD9C7?method=download&shareKey=4c2a2a39e349fa61e68d1d74daffc920)

> 没啥可说的，就两个字儿，太丑。这个是肯定得优化的，我做法是在上面罩一个Button,然后把button的宽度设置成100%,透明度设置为0。贴一下代码。

```html
<button class="upload-btn btn btn-primary"">
    <span class="glyphicon glyphicon-upload"></span>
    上传图片
</button>
<input type="file" class="upload-input-file" accept="image/gif,image/jpeg,image/jpg,image/png,image/svg" id="file_upload" name="file" />
```
```css
input[type=file] {
      margin-top: 5px;
      opacity: 0;
      position: absolute;
      top:0;
      width: 100%;
      height: 100%;
    }
```

>* ### 设置accept属性
> 因为只接受图片格式，所以设置input 的accept属性。
>
>通常做法是设置accept属性为image/*,
```html
<input type="file" class="upload-input-file" accept="image/*" />
```
>
>用之后发现在PC上点击上传照片时，响应非常慢
>
>改！搜了一下发现accept属性要分开写，所以改成了酱
```html
<input type="file" class="upload-input-file" accept="image/gif,image/jpeg,image/jpg,image/png,image/svg" >
```
>
>再用之后发现会导致在Android的部分机型的微信浏览器里上传图片时，不能使用拍照功能。
>
>改！初始时accept还是分开写，然后判断当前浏览器环境，如果在微信里的话再改accept属性

```js
var u = navigator.userAgent;
if(u.toLowerCase().match(/MicroMessenger/i)=='micromessenger'){
    $('.upload-input-file').attr('accept','image/*');
}

```
> 酱貌似没什么问题了,当时搜解决方案的时候还有看到说需要设置capture属性为camera的，我也试过，但是好像会导致IOS只能拍照不能从相册中取图。建议大家一定要多尝试尝试，多测测各种手机和各种浏览器。

>* ### 图片压缩
> 图片上传如果传原图的话上传时间真的坑坑的。需要进行图片压缩。大多数时候前端只需要把图片转成base64码，然后把base64传给后台就行了。但是这次的后台只接受file类型的数据，所以我需要图片--->base64--->压缩----->图片。
>
>1. 图片--->base64: 直接利用FileReader对象的readAsDataURL，将file对象转成base64的格式,在onload回调函数中通过this.result来获取转化的结果
>
> ```js
>var reader = new FileReader();
>    reader.readAsDataURL(file);
>
>    reader.onload = function(e) {
>    console.log(this.result)
>   }
> ```
>
>2. base64--->压缩: 利用canvas，canvas的toDataURL方法可以根据指定参数压缩图片质量，这个代码跟后面的一起贴
>
>
>3. 压缩--->图片：这个部分是get到的新知识。利用到blob，一个Blob对象表示一个不可变的, 原始数据的类似文件对象。Blob表示的数据不一定是一个JavaScript原生格式。 File 接口基于Blob，继承 blob功能并将其扩展为支持用户系统上的文件。
>
>```js
>//将base64 转化为blob
>function dataURLtoBlob(dataurl) {
>          var arr = dataurl.split(','), mime = arr[0].match(/:(.*?);/)[1],
>         bstr = atob(arr[1]), n = bstr.length, u8arr = new Uint8Array(n);
>          while(n--){
>              u8arr[n] = bstr.charCodeAt(n);
>          }
>          return new Blob([u8arr], {type:mime});
>      }
>```
>* ### 图片旋转的Bug
>IOS拍照的时候会把图片旋转90°，这是IOS的bug，这样的图片传到后台的话就会影响到识别功能，所以需要在前端修复这个IOS的bug。
>
> (1) 第一种解决方案就是利用exif，利用exif的getData方法,如下面代码，取到的orientation就是拍照的方法，然后对不同orientation，进行不同的旋转,也就是getImgData方法，其中getImgData还涉及到上面提到的用canvas来压缩图片的操作

```js
    EXIF.getData(file, function () {
        orientation = EXIF.getTag(this, "Orientation");
    });
    
    /**
       *  修正 IOS 拍照上传图片裁剪时角度偏移（旋转）问题
       *
       *  img 图片的 base64
       *  dir exif 获取的方向信息
       *  next 回调方法，返回校正方向后的 base64
       * 
      **/
      function getImgData(img, dir, next) {
       
          var image = new Image();
       
          image.onload = function () {
       
              var degree = 0, drawWidth, drawHeight, width, height;
       
              drawWidth = this.naturalWidth;
              drawHeight = this.naturalHeight;
       
              // 改变一下图片大小
              var maxSide = Math.max(drawWidth, drawHeight);
       
              if (maxSide > 600) {
                  var minSide = Math.min(drawWidth, drawHeight);
                  minSide = minSide / maxSide * 600;
                  maxSide = 600;
                  if (drawWidth > drawHeight) {
                      drawWidth = maxSide;
                      drawHeight = minSide;
                  } else {
                      drawWidth = minSide;
                      drawHeight = maxSide;
                  }
              }
       
              // 创建画布
              var canvas = document.createElement("canvas");
              canvas.width = width = drawWidth;
              canvas.height = height = drawHeight;
              var context = canvas.getContext("2d");
       
              // 判断图片方向，重置 canvas 大小，确定旋转角度，iphone 默认的是 home 键在右方的横屏拍摄方式
              console.log("dir"+dir);
              switch (dir) {
                  // iphone 横屏拍摄，此时 home 键在左侧
                  case 3:
                      // alert(3);
                      degree = 180;
                      drawWidth = -width;
                      drawHeight = -height;
                      break;
                  // iphone 竖屏拍摄，此时 home 键在下方(正常拿手机的方向)
                  case 6:
                      // alert(6);
                      canvas.width = height;
                      canvas.height = width;
                      degree = 90;
                      drawWidth = width;
                      drawHeight = -height;
                      break;
                  // iphone 竖屏拍摄，此时 home 键在上方
                  case 8:
                      // alert(8);
                      canvas.width = height;
                      canvas.height = width;
                      degree = 270;
                      drawWidth = -width;
                      drawHeight = height;
                      break;
                  default:break;
              }
       
              // 使用 canvas 旋转校正
              context.rotate(degree * Math.PI / 180);
              context.drawImage(this, 0, 0, drawWidth, drawHeight);
       
              // 返回校正图片
              next(canvas.toDataURL("image/jpeg", .8));
          }
       
          image.src = img;
       
      }
    
    getImgData(e.target.result, orientation, function (data) {
        console.log(data);
    }
```
> (2) 上面呢种方法有一个缺点就是安卓的使用exif的getData返回结果特别慢，所以用了另一种方法来计算orientation，这种方法兼容IOS和安卓，而且速度快很多
```js
function getOrientation(file, callback) {
        var reader = new FileReader();
        reader.onload = function(e) {

          var view = new DataView(e.target.result);
          if (view.getUint16(0, false) != 0xFFD8) return callback(-2);
          var length = view.byteLength, offset = 2;
          while (offset < length) {
            var marker = view.getUint16(offset, false);
            offset += 2;
            if (marker == 0xFFE1) {
                if (view.getUint32(offset += 2, false) != 0x45786966) return callback(-1);
                var little = view.getUint16(offset += 6, false) == 0x4949;
                offset += view.getUint32(offset + 4, little);
                var tags = view.getUint16(offset, little);
                offset += 2;
                for (var i = 0; i < tags; i++)
                  if (view.getUint16(offset + (i * 12), little) == 0x0112)
                    return callback(view.getUint16(offset + (i * 12) + 8, little));
            }
            else if ((marker & 0xFF00) != 0xFF00) break;
            else offset += view.getUint16(offset, false);
          }
          return callback(-1);
        };
        reader.readAsArrayBuffer(file.slice(0, 64 * 1024));
      }
```
> * ### IOS上传图片会增大图片的大小
>
>因为识别的图片不能过大也不能过小，因此需要判断图片的大小。一开始是后台读取图片大小，并且只读取后台的返回值就行了。但是后来发现在IOS上面，25K的图传上去就变成35K，后台就会判断这个图可以识别，于是就会给我错误的返回值，这就影响到功能了。So.......在前端需要判断当前环境是IOS还是安卓，如果是IOS，把当前file.size-10*1024得到的就是图片的实际大小。然后前端直接判断图片是否超出识别范围，超出的话直接返回错误就行，就不用坑爹的后台了。。。。

>* ### 服务器上的js更新后再安卓上不加载。。。。
>这个错我真的超无语。。。。。。醉了好吗。。。。为什么不加载！！！我也不知道！！！后来听mentor小姐姐的话乖乖给js后面添加了一个时间戳。。又get了一招。
