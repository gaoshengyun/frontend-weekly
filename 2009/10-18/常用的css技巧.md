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