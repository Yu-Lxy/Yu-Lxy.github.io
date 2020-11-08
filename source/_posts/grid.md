---
title: Grid网格布局常用属性总结
date: 2020-11-08 20:39:29
tags: css
categories: css
---

Grid 布局与 Flex 布局有一定的相似性，都可以指定容器内部多个项目的位置。但是，它们也存在重大区别。

Flex 布局是轴线布局，只能指定"项目"针对轴线的位置，可以看作是一维布局。Grid 布局则是将容器划分成"行"和"列"，产生单元格，然后指定"项目所在"的单元格，可以看作是二维布局。Grid 布局远比 Flex 布局强大。

以上两段话都是阮一峰大神说的哈~😆
本篇主要记录一些主要的概念和常用的属性😏

<!-- more -->
### 概念
#### 容器和项目
> 采用网格布局的区域，称为”容器“（container）。容器内部采用网格定位的子元素，称为”项目“（item）。

#### 行和列
> 容器里面的水平区域称为”行“（row），垂直区域称为”列“（column）。

#### 单元格
> 行和列交叉区域，称为”单元格“（cell）。

#### 网格线
> 划分网格的线，称为”网格线“（grid line）。水平网格线划分出行，垂直网格线划分出列。

### 容器属性
 - `display: grid`  指定一个容器采用网格布局
 - `display: inline-grid` 指定该元素为行内元素，内部采用网格布局
 - `grid-template-columns：10px 20px 30%` 列宽，每个值和每列一一对应，可像素或百分数
 - `grid-template-rows：10px 20px 30%` 行高，每个值和每行一一对应，可像素或百分数
 - `repeat()` 简化重复的值，例：grid-template-rows: repeat(3, 100px);
 - `auto-fill关键字` 自动填充个数，例：grid-template-columns: repeat(auto-fill, 100px);
 - `fr关键字` 比例关系，例：grid-template-columns: 1fr 1fr;
 - `minmax()` 长度范围，例：grid-template-columns: 1fr 1fr minmax(100px, 1fr);
 - `auto关键字` 自定宽度，例：grid-template-columns: 100px auto 100px;
 - `[]可定义网格线名字` 例：grid-template-columns: [c1] 100px [c2] 100px [c3] auto [c4];
 - `row-gap:10px` 行间距
 - `column-gap: 20px` 列间距
 - `gap: 10px 20px` 行间距和列间距的合并写法
 - `grid-auto-flow: row` 排列顺序，先行后列
 - `grid-auto-flow: row dense` 排列顺序，先行后列，紧密填满
 - `grid-auto-flow: column` 排列顺序，先列后行
 - `grid-auto-flow: column dense` 排列顺序，先列后行，紧密填满
 - `justify-items: start | end | center | stretch;` 设置单元格内容的水平位置（左中右）
 - `align-items: start | end | center | stretch;` 设置单元格内容的垂直位置（上中下）
 - `place-items: <align-items> <justify-items>;` align-items属性和justify-items属性的合并简写形式。例：place-items: start end;
 - `justify-content: start | end | center | stretch | space-around | space-between | space-evenly;` 整个内容区域在容器里面的水平位置（左中右）
 - `align-content: start | end | center | stretch | space-around | space-between | space-evenly;` 整个内容区域的垂直位置（上中下）
 - `place-content: <align-content> <justify-content>` align-content属性和justify-content属性的合并简写形式。

### 项目属性
 - `grid-column-start: 2` 指定项目的左边框所在的垂直网格线
 - `grid-column-end: 3` 指定项目的右边框所在的垂直网格线
 - `grid-row-start: 4` 指定项目的上边框所在的水平网格线
 - `grid-row-end: 5` 指定项目的下边框所在的水平网格线
 - `span关键字` 表示”跨越“，即左右边框（上下边框）之间跨越多少个网格，例：grid-column-start: span 2;
 - `grid-column 属性` 是grid-column-start和grid-column-end的合并简写形式，例：grid-column: 1 / 2;
 - `grid-row 属性` 是grid-row-start属性和grid-row-end的合并简写形式，例：grid-row: 1 / 3;
 - `grid-area 属性` 指定项目放在哪一个区域，例：grid-area: e;
 - `justify-self: start | end | center | stretch` 设置单元格内容的水平位置（左中右）
 - `align-self: start | end | center | stretch` 设置单元格内容的垂直位置（上中下）
 - `place-self: <align-self> <justify-self>` 是align-self属性和justify-self属性的合并简写形式