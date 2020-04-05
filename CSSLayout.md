# CSS 布局

## 参考资料
- [Learn CSS Layout](http://learnlayout.com)
- [MDN CSS](https://developer.mozilla.org/en-US/docs/Web/CSS)

## 几个重要的属性 property
### 1. display 属性
> 建议直接参考：[MDN-display](https://developer.mozilla.org/en-US/docs/Web/CSS/display)

该属性用来设置元素内部 inner 和外部 outer 的显示类型 display types：
- 外部显示类型：即设置该元素在流布局中的表现（主要是 block 和 inline）
- 内部显示类型：

- none 可用来隐藏元素（真隐藏）
    > `visibility: hidden;` 虽隐藏但占据原来的空间
- block 块级元素
- inline 行内元素
- inline-block 行内块元素
    
    可以替代使用 `float: left;` 实现的多元素自动从左到右，从上到下的排列，但后面的元素不需要设置 `clear: left;`
    > IE6 和 IE7 支持所需的[额外工作](https://blog.mozilla.org/webdev/2009/02/20/cross-browser-inline-block/)

- flex 弹性

### 2. position 属性
默认值为 `static`, 在这里 `position != static` 的元素被称为 **`positioned element`**，可理解为：需要设置定位的元素。
- static 默认值
    此时使用 left, right, bottom, top, z-index 属性无效
- relative 相对
    1. 在没设置 `top, rigth, bottom, left` 属性之前，表现得和 static 一样
    2. 在设置了 `top, rigth, bottom, left` 属性之后，元素会偏离正常位置
    3. 但是该元素在文档流中占据的位置不会因偏离而改变
        > 即文档流中会保留一个设置 top/right/bottom/left 之前的位置
- absolute 绝对
    1. **相对于最近的 positioned 祖先元素 `nearest positioned ancestor` 来定位，如果没有，则相对于 body 元素**
    2. 可使用 `top, rigth, bottom, left` 属性改变元素的位置

    > 良好实践：在需要设置 `display: absolute;` 属性的元素外面添加一个 `display: relative;` 的元素
- fixed 固定
    1. 相对于浏览器窗体来定位
    2. 可使用 `top, rigth, bottom, left` 属性改变元素的位置

    > 注意：移动端浏览器对 fixed 的支持较差，[解决方案(需要番羽土啬)](http://www.bradfrostweb.com/blog/mobile/fixed-position/)

总结：relative, absolute, fixed 都可以达到我们广义的浮动效果，但依据的对象不同
- relative: 相对文档流中的自己
- absolute: 相对最近的 positioined ancestor element
- fixed: 相对于浏览器窗体

> 关于相对定位和绝对定位可以参考 [这个例子](https://www.cnblogs.com/heroine/p/5852748.html)，有实例（最后 `.cart_btn span i` 我用 `displau: absolute;` 完成）

### 3. float 属性
设计初衷：实现文字环绕图片的效果

#### 清除浮动
1. 使用 clear 属性来简单清除
可在浮动元素后面的元素设置 `clear: left/right/both;` 来清除浮动元素的影响
> clear 的值取决于 float 的值。一般来说使用 `both` 就行
2. 使用 clearfix hack 技术

场景：浮动图片高度比包含其的元素要大，此时图片溢出到元素外

解决：在包含图片的元素中添加以下属性
``` css
.clearfix {
  overflow: auto;
  zoom: 1; /* 支持 IE6 时需要 */
}
```
3. 其他技术
参考 [这里](https://stackoverflow.com/questions/211383/what-methods-of-clearfix-can-i-use)

## 媒体查询 media-query
可以实现响应式设计 Responsive Design
``` css
/* 当 width >= 600px 时，按如下设置 */
@media screen and (min-width:600px) {
  nav {
    float: left;
    width: 25%;
  }
  section {
    margin-left: 25%;
  }
}
/* 当 width <= 599px 时，按如下设置 */
@media screen and (max-width:599px) {
  nav li {
    display: inline; /* 导航栏变成行内元素 */
  }
}
```

> [MDN-Using media queries](https://developer.mozilla.org/en-US/docs/Web/CSS/Media_Queries/Using_media_queries)
>
> 使用 [meta viewport](https://dev.opera.com/articles/an-introduction-to-meta-viewport-and-viewport/) 之后可以让你的布局在移动浏览器上显示的更好。 


## 案例
### 1. 使用各种方式实现左导航栏，右侧主题的效果
``` css
/* 1. display: absolute; 实现 */
.container {
  position: relative;
}
nav {
  position: absolute;
  left: 0px;
  width: 200px;
}
section {
  /* position is static by default */
  margin-left: 200px;
}

/* 2. float: left; 实现 */
nav {
  float: left;
  width: 200px;
}
section {
  margin-left: 200px;
}

/* 3. 百分比宽度实现 */
nav {
  float: left;
  width: 25%; /* 页面缩小时有问题，设置 min-width/max-width 也无济于事，因 margin-left: 25%;*/
}
section {
  margin-left: 25%;
}

/* 4. display: inline-block; 实现 */
nav {
  display: inline-block;
  vertical-align: top;
  width: 25%; /* 需要为每列设置宽度 */
}
.section {
  display: inline-block;
  vertical-align: top;
  width: 75%; /* 需要为每列设置宽度 */
}
```

### 2. div 水平居中

> 设置宽度 + 水平 margin 设置为 auto

``` css
#main {
  max-width: 600px; /* 设置最大宽度可以在浏览器窗口小于 600px 时自动缩放 */
  margin: 0 auto; 
}
```