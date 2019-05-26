#### 移动端点击300ms延迟以及fastclick原理

+ 问题的出现

  2007年苹果公司解决用户,是否是双击,会在一次点击内停留300ms.如果用户没有继续点击.就执行单击操作,如果再次点击就执行双击缩放操作

  那会手机是支持缩放的屏幕的

+ 解决方法

  + ```js
    禁止缩放
    	user-scalable=no
    ```

​	      方法二:引包

```html
<!-- 1. 引入一个专门解决点击事件延迟的包 -->
 <script src="./fastclick.js"></script>
```

方法三:手写封装函数

```js
	第一步:假如一次触摸 只触发了touchstart 和 touchend没有触发touchmove 就认为是单击操作
        // 如果触发了touchmove就不是
		开关思想结合起来
```

#### fastclick原理

```js
   移动端，当用户点击屏幕时，会依次触发 touchstart，touchmove(0 次或多次)，touchend，mousemove，mousedown，mouseup，click。 touchmove 。
  只有当手指在屏幕发生移动的时候才会触发 touchmove 事件。在 touchstart ，touchmove 或者 touchend 事件中的任意一个调用 event.preventDefault，mouse 事件 以及 click 事件将不会触发。
  fastClick 在 touchend 阶段 调用 event.preventDefault，然后通过 document.createEvent 创建一个 MouseEvents，然后 通过 event​Target​.dispatch​Event 触发对应目标元素上绑定的 click 事件。

```

   1.那么重点就是:当我们手指点击屏幕最开始出发的就是touchstart

2. 只有在屏幕上发生滑动才会触发 touchmove 事件

3.  fastClick 在 touchend 阶段 调用 event.preventDefault，阻止 mouse 事件 以及 click 事件的默认行为

4. 将click 事件绑定到touch时间里面

5.  touch 包含三类事件，它们分别是：touchstart、touchmove、touchend 。当你的手指点击屏幕的时候,一定会触发至少是touchstart , touchend ,有可能触发touchmove

   ```js
   touchstart：手指触摸到一个 DOM 元素时触发。
   
    
   touchmove：手指在一个 DOM 元素上滑动时触发。
   
    
   touchend：手指从一个 DOM 元素上移开时触发。
   
   那么获取屏幕手指信息,怎么获取呢
   	当用户手指放在移动设备在屏幕上滑动会触发的touch事件
   	touchstart
   		当手指触摸屏幕时发生,不管当前有多少手指
   	touchmove
   		当手指在屏幕上滑动时连续触发。通常我们再滑屏页面，会调用event的preventDefault()可以阻止默认情况的发生：阻止页面滚动
   	touchend
   		当手指离开屏幕时触发
   	touchcancel
   		系统停止跟踪触摸时候会触发。例如在触摸过程中突然页面alert()一个提示框，此时会触发该事件，这个事件比较少用
   	touchEvent事件:
   		touches
   			屏幕上所有手指的信息
   		targetTouches
   			手指在目标区域的手指信息
   		changedTouches
   			最近一次触发该事件的手指信息
   		touchend时，touches与targetTouches信息会被删除，changedTouches保存的最后一次的信息，最好用于计算手指信息
   
   ```

   

#### 注意

```js
首先,300ms 的延迟只有在移动端才会出现，PC 端是没有的。fastClick 中又有个一 notNeeded 的函数是用来判断有没有必要使用 fastClick。刚开始的时候，刚开始我阅读完代码表示对没有进行移动端和 PC 端的区分表示不满。不过后来一段不起眼的代码改变了我的看法。

// Devices that don't support touch don't need FastClick
if (typeof window.ontouchstart === 'undefined') {
  return true;
}
```

1. 核心点就是pc端是没有touch事件的  , PC网页上的大部分操作都是用鼠标的，即响应的是鼠标事件，包括`mousedown`、`mouseup`、`mousemove`和`click`事件。一次点击行为，可被拆解成：`mousedown` -> `click` -> `mouseup`三步。
2. 因此 `window.ontouchstart` 返回 `undefined`，移动端如果没有绑定事件则返回 `null`。

这里我们还要考虑一下click事件的兼容性问题

1. Internet Explorer 8 & 9存在一个漏洞，具有经[`background-color`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/background-color)样式计算为[`transparent`](https://developer.mozilla.org/en-US/docs/Web/CSS/color_value#transparent_keyword)的元素覆盖在其它元素顶端时，不会收到`click`事件。取而代之，所有`click`事件将被触发于其底下的元素。

   ```
   已知会触发此漏洞的情景：
   
   仅对于IE9：
   设置background-color: rgba(0,0,0,0)
   设置opacity: 0 并且明确指定background-color而不是transparent
   对于IE8和IE9：设置filter: alpha(opacity=0);并且明确指定background-color而不是transparent
   ```

2. ### Safari手机版

   ```
   safari手机版会有一个bug，当点击事件不是绑定在交互式的元素上（比如说HTML的div），并且也没有直接的事件监听器绑定在他们自身。
   
   解决方法如下：
   
   为其元素或者祖先元素，添加cursor: pointer的样式，使元素具有交互式点击
   为需要交互式点击的元素添加onclick="void(0)"的属性，但并不包括body元素
   使用可点击元素如<a>,代替不可交互式元素如div
   不使用click的事件委托。
   Safari 手机版里，以下元素不会受到上述bug的影响：
   
   <a> 需要href链接
   <area> 需要href
   <button>
   <img>
   <input>
   <label> 需要与form控制器连接
   ```

关于事件冒泡 event.stopPropagation

`event.stopPropagation` 只会阻止相同类型(event.type 相同)事件传播，上面有提到过 移动端 触摸事件触发的顺序问题，假如 我在 `touchstart` 中调用了 `event.stopPropagation` 只会 阻止后续 event flow 上其他 `touchstart` 事件，并不会阻止 `touchmove`，`touchend` 等 mouseEvent 事件的发生。

`event.stopPropagation`，`event.stopImmediatePropagation`的区别你真的知道吗 🧐，`event.stopPropagation` 阻止捕获和冒泡阶段中当前事件的进一步传播。如果有多个相同类型事件的事件监听函数绑定到同一个元素，当该类型的事件触发时，它们会按照被添加的顺序执行。如果其中某个监听函数执行 `event.stopImmediatePropagation` 方法，则当前元素剩下的监听函数将不会被执行。

`eventTarget.dispatchEvent` 仍然会触发完整的 event flow，而不仅仅触发 eventTarget 本身注册的事件。





##### 其它

1. audio元素和video元素在ios和andriod中无法自动播放
   	$('html').on('touchstart',function(){
       audio.play()
   })
   	应对方案：触屏即播
2. 手机拍照和上传图片
   	<input type="file">的accept 属性
   	<!-- 选择照片 -->
   <input type=file accept="image/*">
   <!-- 选择视频 -->
   <input type=file accept="video/*">

3. 加边框问题
   	第一种方案
   		加内边框,盒子不会放大,内容会被压缩
   		box-sizing: border-box;
   	第二种
   		加外边框 内容不会被压缩,但是边框会放大
   		box-sizing: content-box;
   	第三种
   		利用盒子阴影,添加边框 . 不会把内容和边框改大或改小
   			.top {  box-shadow: 0 -2px 0 red;   }  
   .right {  box-shadow: 2px 0 0 green;   } 
   .bottom {  box-shadow: 0 2px 0 blue;  }  
   .left {  box-shadow: -2px 0 0 orange;  }  
   	第四种
   		伪元素添加边框  不会把内容和边框改大或改小
   		但是需通过定位,顶到指定 指定位置
4. 页面滚动的位置信息判断
   	 分类页面的滑动
               1. 使用原生滑动事件  touchstart touchmove touchend
                           2. e.touches[0].clientX/Y 如果在end里面 只能使用 changedTouches[0].clientX/Y   因为在touchend 里,只会保留changed 最近一次的手指信息
   	 // touchend结束事件因为这个时候屏幕和页面上都没有手指了 touches targetTouches拿不到
                           // 必须使用 changedTouches 改变的触摸对象 手指松开了是一个改变就能获取到