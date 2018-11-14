## 原生JavaScript实现轮播图
### HTML和CSS部分
将banner这个div分为两栏，采用float实现左右布局，左边展示大图片，右边展示小图片和一个滑块背景。
左边和右边都有设置了相对定位（position:relative)的容器div,图片作为列表项放在无序列表中。
其中，左边包含大图片的列表项设置绝对定位 重叠在一起，通过设置z轴z-index和透明度opacity来显示要展示的图片。
右边的滑块背景设置绝对定位以便使用margin-top属性来定位及后面的滑动效果，图片则按照列表默认的垂直布局展示。
以上为HTML和CSS的大致思路，细节处理请看代码

### JavaScript部分
JavaScript则是实现轮播图的核心部分，我们来详细分析一下JavaScript代码。

#### 大致思路
轮播图大致可以分为两个功能模块
- 鼠标事件交互
- 自动轮播

我们为右边的每个小图片和滑块背景都添加上事件监听。
当鼠标移入小图片时，将对应的大图片的透明度和z-index设为1，将其他大图片的透明度设为0，z-index设为-2，并实现滑块背景的移动
当鼠标移出时，调用auto()函数启动自动轮播

接下来我们来分模块详细分析一下功能实现。



#### 通过DOM获取html元素
//将以下代码放入window.onload或者把js文件在body底部引用，不然会获取不到元素
var bannerLeft = document.getElementsByClassName("bannerLeft");
var bannerRight = document.getElementsByClassName("bannerRight");
var hoverBg = document.getElementsByClassName("hover-bg");

DOM是JavaScript操控html元素的关键，毕竟你要操控html元素，需要先获取他们。
注：getElementsByClassName返回的是包含所有对应class的元素的数组。


#### 为元素添加事件监听

function hover(){    
    // 事件监听要用匿名函数传参数
    bannerRight[0].addEventListener("mouseover",function(){mouseOver(0);clearInterval(auto_timer);});
    bannerRight[1].addEventListener("mouseover",function(){mouseOver(1);clearInterval(auto_timer);});
    bannerRight[2].addEventListener("mouseover",function(){mouseOver(2);clearInterval(auto_timer);});
    bannerRight[3].addEventListener("mouseover",function(){mouseOver(3);clearInterval(auto_timer);});
    bannerRight[4].addEventListener("mouseover",function(){mouseOver(4);clearInterval(auto_timer);});
    hoverBg[0].addEventListener("mouseover",function(){clearInterval(auto_timer);});
    
    bannerRight[0].addEventListener("mouseout",function(){flag = 1;auto();});
    bannerRight[1].addEventListener("mouseout",function(){flag = 2;auto();});
    bannerRight[2].addEventListener("mouseout",function(){flag = 3;auto();});
    bannerRight[3].addEventListener("mouseout",function(){flag = 4;auto();});
    bannerRight[4].addEventListener("mouseout",function(){flag = 0;auto();});
    hoverBg[0].addEventListener("mouseout",function(){flag = switch_target/60 + 1;if(flag == 5)flag=0;auto();});
}

这个hover()函数是我们需要调用的总函数，为元素添加上了事件监听
其中flag是我为了确定在自动轮播功能中确定滑块背景位置设置的全局变量
注：当时为了快速实现功能，没有考虑全局变量带来的危害，本代码中大量使用了全局变量

#### mouseOver()函数

function mouseOver(i){
    // (function(i){
        for(let j = 0; j < 5; j ++){
            if(j == i){
                getHover(j);
            }else{
                loseHover(j);
            }
        }
    // })(i);
}
该函数接受一个变量i(确定鼠标移入的是哪个图片),对包含左边大图片的数组进行遍历，如果是对应的大图片则调用getHover()函数，否则调用loseHover()函数，这个两个函数也接受一个变量，来确定对应图片


#### getHover()函数

function getHover(i){
    bannerLeft[i].style.zIndex = "1"; //设置z-index
    opacity(1,i); //调用函数设置透明度，为了实现透明度渐变特效，我们抽离出一个opacity()函数来实现功能
    blockSwitch(i); //调用blockSwitch()函数来实现滑块背景的滑动
    flag = i; // 设置flag
}


#### loseHover()函数

function loseHover(i){
    bannerLeft[i].style.zIndex = "-2";  
    bannerLeft[i].style.opacity = 0;
}


#### 实现透明渐变 opacity(target,i)函数

var opacity_alpha = [1,0,0,0,0];    //存放对应图片的透明度
var opacity_speed = 0;  //渐变速度，通过设置正负来控制透明度增加还是减少，注：上面的轮播中只使用了透明度增加，因为淡入和淡出同时存在效果不是很好，就没用淡出，懒得改代码这个函数就保持原样了。
var opacity_timer = null; // 计时器
function opacity(target,i){
    //将其他图片的透明度设为0
    if(switch_target/60 != i)
        opacity_alpha[i] = 0;
    clearInterval(opacity_timer); // 每次调用前先清除之前的定时器
    opacity_timer = setInterval(function(){
        if(target > opacity_alpha[i]){
            opacity_speed = 0.03;
        }else if(target < opacity_alpha[i]){
            opacity_speed = -0.03;
        }
        // 当透明度达到目标值，清除定时器，并把速度设为0，
        // 否则，更新图片的透明度
        // 此处用opacity_alpha[i] > target - 0.01是因为当时写代码的时候用==不能停止计时器，可能是小数运算的坑，没有深入研究
        if(opacity_alpha[i] > target - 0.01){
            opacity_speed = 0;
            clearInterval(opacity_timer);
        }else{
            opacity_alpha[i] += opacity_speed;   
            bannerLeft[i].style.opacity = opacity_alpha[i];
        }
    },30);
}


#### 实现滑块滑动 blockSwitch(i)函数

var switch_marginTop = 0;
var switch_speed = 0;
var switch_timer = null;
var switch_target = 0;
var switchBlock = document.getElementsByClassName("hover-bg");
function blockSwitch(i){
    switch_target = 60 * i;
    clearInterval(switch_timer);
    switch_timer = setInterval(function(){
        if(switch_target > switch_marginTop){
            switch_speed = 5;
        }else if(switch_target < switch_marginTop){
            switch_speed = -5;
        }
        if(switch_marginTop == switch_target ){
            switch_speed = 0;
            clearInterval(switch_timer);
        }else{
            switch_marginTop += switch_speed;
            //注意 + "px"          
            switchBlock[0].style.marginTop = switch_marginTop + "px";
        }
    },30);
}

滑块滑动的实现思路与透明度渐变的思路相似
透明度渐变是使用计时器实现透明度的逐渐增加或减少，
滑块滑动则是使用计时器实现marginTop属性的逐渐增加或减少，直到目标位置



#### 自动轮播 auto()函数

var flag = 1;
var auto_timer = null;
function auto(){
    clearInterval(auto_timer);
    auto_timer = setInterval(function(){
        mouseOver(flag);
        flag ++;
        if(flag == 5){
            flag = 0;
        }
    },3000);
}

auto()函数其实就是用计时器定时循环调用mouseOver函数，传入的参数flag

#### 最后
最后别忘了调用函数
hover();
auto();