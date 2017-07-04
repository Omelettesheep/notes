# easyui（一）
> 开始做个mis系统，因为只要给后台搭个框架，所以选择了让后台容易上手的easyui。所以技术栈就是easyui+jquery+bootstrap。

## easyui 引入
> 今天刚刚开始做，先用纯纯的bootstrap写完了登录界面。然后把easyui引入了项目，easyui需要引入的东西还蛮多的，记录一下。这部分的引入参考的右边[http://blog.csdn.net/limlimlim/article/details/10294315](http://blog.csdn.net/limlimlim/article/details/10294315)
>

```js
<!--为了保证兼容性，引入easyui中的jquery版本-->
<script src="../js/jquery-easyui-1.5.2/jquery.min.js"></script>
<!--引入easy.min.js-->
<script src="../js/jquery-easyui-1.5.2/jquery.easyui.min.js"></script>
<!--引入css-->
<link rel="stylesheet" href="../js/jquery-easyui-1.5.2/themes/default/easyui.css"/
<!--引入Icon-->
<link rel="stylesheet" href="../js/jquery-easyui-1.5.2/themes/icon.css"/>
<!--引入国际化文件，以显示中文-->
<script src="../js/jquery-easyui-1.5.2/locale/easyui-lang-zh_CN.js" type="text/javascript"></script>
<!--页面上加上UTF-8编码防止jquery.easyui.min.js内容乱码-->
<meta http-equiv="content-type" content="text/html;charset=UTF-8" />
```

## 关于EASYUI读取不到DATAGRID的JSON文件
>贴一下搜到的解决方法
```js
data-options="singleSelect:true,collapsible:true,data:data,method:'get'">
var data = {"total":28,"rows":[
    {"productid":"FI-SW-01","unitcost":10.00,"status":"P","listprice":36.50,"attr1":"Large","itemid":"EST-1"},
    {"productid":"K9-DL-01","unitcost":12.00,"status":"P","listprice":18.50,"attr1":"Spotted Adult Female","itemid":"EST-10"},
    {"productid":"RP-SN-01","unitcost":12.00,"status":"P","listprice":28.50,"attr1":"Venomless","itemid":"EST-11"},
    {"productid":"RP-SN-01","unitcost":12.00,"status":"P","listprice":26.50,"attr1":"Rattleless","itemid":"EST-12"},
    {"productid":"RP-LI-02","unitcost":12.00,"status":"P","listprice":35.50,"attr1":"Green Adult","itemid":"EST-13"},
    {"productid":"FL-DSH-01","unitcost":12.00,"status":"P","listprice":158.50,"attr1":"Tailless","itemid":"EST-14"},
    {"productid":"FL-DSH-01","unitcost":12.00,"status":"P","listprice":83.50,"attr1":"With tail","itemid":"EST-15"},
    {"productid":"FL-DLH-02","unitcost":12.00,"status":"P","listprice":63.50,"attr1":"Adult Female","itemid":"EST-16"},
    {"productid":"FL-DLH-02","unitcost":12.00,"status":"P","listprice":89.50,"attr1":"Adult Male","itemid":"EST-17"},
    {"productid":"AV-CB-01","unitcost":92.00,"status":"P","listprice":63.50,"attr1":"Adult Male","itemid":"EST-18"}
]}
```
>也就是把json文件中的内容写进页面，然后给data-option添加data属性。把以前的url去掉。

>hummm这部分原理方面还不清楚，是不是加了url之后会自动不读本地而去跨域？不知道诶。明天的接着写这个项目肯定会涉及到这边，以后补上这个坑。

> 2017.7.4
