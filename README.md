# 想到做这个，是因为无意中在github上看到了这一个仓库<https://github.com/lunlunshiwo/ChooseCity>，做的就是一个城市选择控件，是用vue写的，自己也试着模写一遍
      
## 效果如下:  

![Image text](E:\Code\项目\vue-city-select\vcityselect\static\choseCity.gif)

  这个组件的制作基本上与该GitHub上的制作代码基本相同，我自己添加了一些小功能，可以说是仿写的吧，毕竟才疏学浅。

## 1.定位服务
```
// 调用高德地图接口返回定位(node 中服务器向服务器发请求)
//node官网中的http.request中例子没有写怎样把从服务器请求来的数据渲染到请求回来的页面中
//这里是有两种方法的一种是向下面那样在官网原有demo基础上加个回调函数，还有一种方法是
//http.get方法这个可以看看一些博客中的介绍官网当然也行
//附上地址：	https://blog.csdn.net/DeepLies/article/details/52771057
		//http://nodejs.cn/api/http.html#http_http_get_url_options_callback
app.get('/nowcity', function (req, res) {
    function showCityInfo(cb){
  
    const options = {
        hostname: 'restapi.amap.com',
        port: 443,
        path: '/v3/ip?key=自己申请的AK&output=json',
        method: 'GET',
    };

    const req = https.request(options, (res) => {
        let pdata = "";
        /*  console.log(`状态码: ${res.statusCode}`);
        console.log(`响应头: ${JSON.stringify(res.headers)}`); */
        res.setEncoding('utf8');
        res.on('data', (chunk) => {
            pdata += chunk;
        });
        res.on('end', () => {
            cb(pdata);//类型必须是string或者buffer
        });
        
    });
    req.on('error', (e) => {
        console.error(`请求遇到问题: ${e.message}`);
    });


    
    req.end();//必须有 
}   
    showCityInfo(function(msg){
         let data = msg;
         res.end(data);
     });
```



## 2.正则  

话说正则是我最头疼的事情。
```
//正则表达式
exports.getSearchList = (text,list,canSearchSpell)=>{
    const reg1 = /^[a-z,A-Z]+$/g ;//检测是否为字母
    const reg2 = new RegExp(`^${text}`,'g');//匹配以text为开头
    const reg3 = new RegExp(`^[\u4e00-\u9fa5]+$`,'g'); //检测是不是汉字
    const reg4 = new RegExp(`^${text}`,'g');//匹配以汉字为开头
    const reg5 = new RegExp(`^${text}`,'g');//匹配以acronym开头
    let resList = [];
    //当text为拼音时
    if(text.match(reg1) && canSearchSpell){
        for(let i=0,len1=list.length;i<len1;i++){
            for(let j=0,len2=list[i][1].length;j<len2;j++){
                if(list[i][1][j].pinyin.match(reg2)){
                    resList.push(list[i][1][j]);
                }else if(list[i][1][j].acronym.match(reg5)){
                    resList.push(list[i][1][j]);
                }
            }
        }//当text为拼音时
    }else if(reg3.test(text)){
        for(let i=0,len1=list.length;i<len1;i++){
            for(let j=0,len2=list[i][1].length;j<len2;j++){
                if(list[i][1][j].name.match(reg4)){
                    resList.push(list[i][1][j]);
                }
            }
        }
    }
    return resList;
}
```

这里呢有人会说reg2和reg4、reg5不一样吗，直接使用reg2不就可以了吗，但是试过之后就会发现根本不管用，这里我暂时还不能讲出原因来




## 3.this.$refs  

在vue中获取DOM元素，不需要用document.querySelector（".select"）或者getElementById（'.select'）获取这个dom节点，vue为我们提供了ref这个属性，只需要在要操作dom的元素上绑定`ref:a`,在this.$refs.a拿到的就是该元素

```
<div ref="wrapper" class="scroll">
</div>
```

此时this.$refs[wrapper]就代表了这个div。

如果绑定动态的ref

```
<p class="city-title" :ref="allCity[0]">{{allCity[0]}}</p>
```

获取dom的时候`this.$refs[val][0]`


## 4.better-scroll的使用

**使用better-scroll时，需有根节点**  

**css中必须有`overflow:hidden`这个属性**
我这里基本就是照着原文写的

 更多内容可以参考文档http://ustbhuangyi.github.io/better-scroll/doc/#scrollbyx-y-time-easing


## 5.class属性的动态绑定

绑定的值可以用对象表示，也可以用数组表示，支持三目运算符

所有的demo可以去vue官网查询https://cn.vuejs.org/v2/guide/class-and-style.html

这里只说一下我遇到的情况

```
<div class="lists">
        <div v-for="allCity in allCities" :key="allCity[0]" >
            <p class="city-title" :ref="allCity[0]">{{allCity[0]}}</p>
            <p :class="{'city-item':true,'vue-1px-t':index}" v-for="(city,index) in allCity[1]" :key="city.id" @click="changeCity(city.name)">
                {{city.name}}
                <i :class="iconCss(city)" ></i>
                </p>   
        </div>
    </div>
```

在cityList组件中我想实现当我点击某一城市该城市右面会出现选中的图标，而其他的不会出现，这里就用到了三目运算符，只不过我把它封装在一个函数中了

```
//具体实现
//父组件把选中的城市的名字传给子组件
<CityList :choiceCityName="choiceCityName"><CityList>
//子组件
props:{
    choiceCityName:{
        type:String
    }
},
methods:{
    iconCss(city){
        return (this.choiceCityName === city.name) ? 'icon icon-zhengque iconfont' : 'icon icon-zhengque iconfont2'
    }
}
```

最后就设置一下iconfont和iconfont2（hidden）的属性




# 总结：  


本次写该组件回顾了好多知识

1. vue的使用（vue-cli，less的webpack的配置）
2. 学习了2个新库（better-scroll（用来滑动）、fastclick（用来消除点击延迟））
3. 更加熟悉node 以及 axios
4. vue父组件将异步获取的数据传给子组件（watch监听）
5. class绑定动态数据
6. 正则熟悉了一下
7. 了解了一些localstorage
8. 对this.$refs有深入的了解（动态绑定获取动态dom）
9. 数组方法unshift（）把数据放到数组第一位，splice(0,2)截取0-2索引位的数组元素但不包括2索引位，findIndex() 方法返回传入一个测试条件（函数）符合条件的数组第一个元素位置。|||字符串方法substring(0,2)截取0-2索引位的字符串元素但不包括2索引位，substr(0,2)从索引位0开始，截取2个字符串，

本文只是摹写的原作者的城市选择组件，并在原有的基础上添加诸如选择城市后标题改变、所选城市后面有选中标记，并且优化了部分使用map的逻辑代码，修复了在使用查询功能时查询结果显示不完全的问题。
本人才疏学浅，如有侵权请联系我！
原著地址：https://github.com/lunlunshiwo/ChooseCity