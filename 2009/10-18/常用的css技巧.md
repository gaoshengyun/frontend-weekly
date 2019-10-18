### **解决 inline-block 元素设置 overflow:hidden 属性导致相信元素向下偏移**

```
.wrap {
  display: inine-block;
  overflow: hidden;
  vertical-align: bottom;
}
```

---

### **超出部分显示省略号**

```
// 单选文本
.wrap {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

// 多行文本
.wrap {
  width: 100%;
  overflow: hidden;
  display: -webkit-box;
  -webkit-box-origent: vertical;
  -webkit-line-clamp: 3; /*限制一个块元素中显示的文本的行数*/
  word-berak: break-all;
}
```

---

### **css 实现不换行, 自动换行, 强制换行**

```
// 不换行
.wrap {
  white-space: nowrap;
}

// 自动换行
.wrap {
  word-wrap: break-word;
  word-break: normal;
}

// 强制换行
.wrap {
  word-berak: break-all;
}
```

---

### **css 实现文本两端对齐**

```
.wrap {
  text-align: justify;
  text-align: distribute-all-lines; // ie6-8
  text-align-last: justify; // 一个块或行的最后一行对齐方式
}
```

### **实现文字竖向排版**

```
// 单列展示
.wrap {
  with: 25px;
  line-height: 18px;
  height: auto;
  font-size: 12px;
  padding: 8px 5px;
  word-wrap: break-word; /*英文的时候需要加上实现自动换行*/
}

//多列展示
.wrap {
  height: 210px;
  line-height: 30px;
  text-align: justify;
  writing-mode: vertical-lr; // 从左向右
  writing-mode: tb-lr;  // IE从左向右
  writing-mode: vertical-rl; // 从右向左
  writing-mode: tb-rl;  // IE从右向左
}
```

---

### **使元素鼠标事件失效**

```
.wrap {
  // 如果按tab能选中该元素, 如button, 然后按驾车还是能执行对应的事件,如click
  pointer-event: none;
  cursor: default;
}
```

---

### **禁止用户选择**

```
.wrap {
  -webkit-touch-callout: none;
  -webkit-user-select: none;
  -khtml-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
}
```

---

### **cursor属性**

```
.wrap {
  cursor: pointer; // 小手指
  cursor: help; // 箭头加问号
  cursor: wait; // 转圈圈
  cursor: move; // 移动光标
  cursor: crosshair;  // 十字光标
}
```

---

### **使用硬件加速**

```
.wrap {
  transform: translateZ(0)
}
```

---

### **图片宽度自适应**

```
img{
  max-width: 100%
}
```

---

### **Text-transform 和 Font Variant**

```
p {
  text-transform: uppercase;  // 将所有字母变成大写字母
  text-transform: lowercase;  // 将所有字母变成小写字母
  text-transform: capitalize; // 首字母大写
  font-variant: small-caps; // 将字体变成小型的大写字母
}
```

---

### **将一个容器设为透明**

```
.wrap {
  filter: alpha(opacity=50);
  opacity: .5;
}
```

---

### **消除 transition 闪屏**

```
.wrap {
  transform-style: preserve-3d;
  backface-visibility: hidden;
  perspective: 1000;
}
```

### **自定义滚动条**

```
overflow: scroll
// 整个滚动条
::-webkit-scrollbar {
  width: 5px;
}

// 滚动条轨道
::-webkit-scrollbar-track {
  background-color: #ffa336;
  border-radius: 5px;
}

// 滚动条滑块
::-webkit-scrollbar-thumb {
  background-color: #ff0c76;
  border-radius: 5px;
}
```

---

### **让 HTML 识别 string 里的 '\n'并换行**

```
body {
  white-space: pre-line;
}
```

---

### **实现一个三角形**

```
.wrap {
  border-color: transparent transparent green transparent;
  border-style: solid;
  border-width: 0 300px 300px;
  height: 0;
  width: 0
}
```

---

### **移除被点链接的边框**

```
a {
  outline: none;
}
```

---

### **使用css显示链接之后的url**

```
a:after{
  content: "(" attr(href) ")";
}
```

---

### **select 内容居中显示, 下拉内容右对齐**

```
select {
  text-align: center;
  text-align-last: center;
}
select option {
  direction: rtl;
}
```

---

### **修改 input 输入框中 placeholder 默认字体样式**

```
// webkit内核浏览器
input::-webkit-input-placeholder {
  color: #f00;
}

// firfox 19+ 
input::-moz-placeholder {
  color: #f00;
}

// ie浏览器
input:-ms-input-placeholder {
  color: #f90;
}
```

---

### **子元素固定宽度父元素宽度被撑开**

```
// 父元素下的子元素是行内元素
.wrap {
  white-space: nowrap;
}

// 若父元素下的子元素是块级元素
.wrap {
  white-space: nowrap;
  display: inline-block;
}
```

---

### **让 div 里的图片和文字同时上下居中**

```
.wrap {
  height: 100px;
  width: 100px;
}
img {
  vertical-align: middle;
}
```

> vertocal-align 用来指定行内元素或者表格单元格元素的垂直方式.只对行内元素, 表格单元格元素生效, 不能用它垂直对齐块级元素.

---

### **宽高等比例自适应矩形**

```
// css code
.scale {
  width: 100%;
  padding-bottom: 56.25%;
  height: 0;
  position: relative;
}
.item {
  position: absolute;
  width: 100%;
  height: 100%;
  background-color: #499e56;
}

// html code
<div class="scale">
  <div class="item">
    这里是所有子地鲜红的容器
  </div>
</div>
```

---

### **transform 的 rotate 属性的在 span 标签下失效**

```
span {
  display: inline-block;
}
```

---

### **边框字体同色**

```
.wrap {
  width: 200px;
  height: 200px;
  color: #000;
  font-size: 30px;
  border: 50px solid currentColor;
}
```