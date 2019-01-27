## 理解 CSS Flexbox

> 原文：[Understanding CSS Flexbox](https://codeburst.io/understanding-css-flexbox-d6162885fefe)
>
> 译者：[neal1991](https://github.com/neal1991)
>
> welcome to star my [articles-translator](https://github.com/neal1991/articles-translator/), providing you advanced articles translation. Any suggestion, please issue or contact [me](mailto:bing@stu.ecnu.edu.cn)
>
> LICENSE: [MIT](https://opensource.org/licenses/MIT)

![](https://cdn-images-1.medium.com/max/2000/1*xpjy9vmoYYR_5RGPBqQoIw.png)

你有没有想过如何最好地使用 flexbox 来排列元素？ 如果你再也不能 忍受 CSS Float hack，让我们深挖一下，并学习如何使用 Flexbox。

![oOLUe.png](https://s1.ax1x.com/2017/12/06/oOLUe.png)

## 什么是 Flexbox?

让我们来注意两个与 Flexbox 属性配合使用的重要元素; Flex 	Container 和 Flex Item。

Flex Container 显然被设置为父元素，而 Flex Items 是 Flex Container 的直接子元素。

因此，Flexbox 布局使得 Flex Container 能够调整 Flex Item 的宽度和高度，以适应各种显示设备和屏幕尺寸的可用空间。

>  如上所述，在 ***flex container*** 中存在一个或者多个 ***flex item***。

## **Flex Container 中的 Flexbox 属性**

1. **Display: Flex**

在下图中，我们有四个框，默认情况下每个框是一个块元素; 也就是说，他们应该占据整行。 Flexbox 可以在所有容器的直接孩子（即四个盒子）上启用 flex 属性，而四个框则以内联方式显示。

![](https://cdn-images-1.medium.com/max/2000/1*hb6tOl-7bpB_xOV2XDG3GA.png)

    .container {
       display: flex;
    }

2. **Flex Direction**

我们经常尝试垂直或水平对齐菜单列表项。 然后，我们必须使用列表项，然后将无序列表的css属性显示为 ***display-inline***。

但是通过Flex Container 的 Flex-direction，它可以帮助我们指定容器内的 Flex Item 的对齐方向（比如从左到右或从右到左）。 以下是 flex-direction 的属性：

***row, row-reverse, column and column-reverse.***

![](https://cdn-images-1.medium.com/max/2000/1*vPkSdz6kf2yFFPLN_0oBYg.png)

​									display: flex;
 								***flex-direction: column;***

    .container {
       display: flex;
       flex-direction: column;    /** lays the Flex Items vertically from top to bottom **/
    }

![](https://cdn-images-1.medium.com/max/2000/1*hb6tOl-7bpB_xOV2XDG3GA.png)

​							display: flex;
 						***flex-direction: row;***

    .container {
       display: flex;
       flex-direction: row; /** Places the Flex Items from left to right **/
    }

3. **Flex Wrap**

让我们假设我们在我们团队的页面上有一个包含12张个人资料照片的画廊，我们可以选择将所有12张图片连续放置，或者将图片包装放在其他行上。

借助 Flex-wrap，我们可以将所有图像保留在一行上，或者轻松地包装到另一行中。 以下是 flex-wrap 的属性：

***nowrap, wrap and wrap-reverse.***

![images are not wrapped to another line and this looks terrible](https://cdn-images-1.medium.com/max/2000/1*10-Z2OdPKwYS3QDVNFfs-Q.png)

​     			       图片没有包装到另一行并且这看起来很糟糕

    .container {
       display: flex;
       flex-wrap: nowrap; /** default property. all profile pictures will be on one line **/
    }

![images will wrap onto new lines](https://cdn-images-1.medium.com/max/2000/1*49fmrP7OosgNa7l6RI_bVQ.png)

​								图片将会包装到新行中

    .container {
        display: flex;   
        flex-wrap: wrap; /** images will wrap onto another line if necessary, from top to bottom **/
    }

4. **Justify Content**

对于 Justify-content 具有以下属性: 

***flex-start, flex-end, center, space-around and space-between.***

![](https://cdn-images-1.medium.com/max/2000/1*DT3irNO28CMlTeEOByVTJQ.png)

    .container {
       display: flex;
       justify-content: flex-start;
       border: 1px solid black;
    }

![](https://cdn-images-1.medium.com/max/2000/1*VoPMWkKRRb-X7X8FfXR7eA.png)

    .container {
       display: flex;
       justify-content: flex-end;
    border: 1px solid black;
    }

![](https://cdn-images-1.medium.com/max/2000/1*FZeechhQD1IVta2G_VJgLg.png)

    .container {
       display: flex;
       justify-content: center;
    border: 1px solid black;
    }

![](https://cdn-images-1.medium.com/max/2000/1*5ZLXFMHJ3C250ZzlovArhw.png)

    .container {
       display: flex;
       justify-content: space-between;
    border: 1px solid black;
    }

![](https://cdn-images-1.medium.com/max/2000/1*xA5jAgh2HzPNYeqXYXDF1w.png)

    .container {
       display: flex;
       justify-content: space-around;
    border: 1px solid black;
    }

**5. Align Items**

就像对齐内容一样，align-items 帮助我们对齐横轴上的flex项目，即从上到下垂直排列，在某些情况下从下到上排列。

我们有以下属性的对齐项目：

***flex-start, flex-end, center, stretch and baseline.***

![cards align on the top of the cross-axis](https://cdn-images-1.medium.com/max/2000/1*qVoeSLvHQf63vZ_cfG7N-g.png)

    .container {
      height: 50vh;
      display: flex;
      flex-wrap: wrap;
      justify-content: center;
      align-items: flex-start;
      border: 1px solid black;
    }

![cards align on the end of the cross-axis](https://cdn-images-1.medium.com/max/2000/1*hD5JeOPEfPPawpixwZzgVA.png)

    .container {
      height: 50vh;
      display: flex;
      flex-wrap: wrap;
      justify-content: center;
      align-items: flex-end;
      border: 1px solid black;
    }

![](https://cdn-images-1.medium.com/max/2000/1*vzpHtRSgzSMCRIzNjP7oGQ.png)

    .container {
      height: 50vh;
      display: flex;
      flex-wrap: wrap;
      justify-content: center;
      align-items: baseline;
      border: 1px solid black;
    }

![cards are of different heights but they still align at the center](https://cdn-images-1.medium.com/max/2000/1*TOWFH-Duvckx2PZRAp9A5A.png)

​							              不同高度的图片仍然居中

    .container {
      height: 50vh;
      display: flex;
      flex-wrap: wrap;
      justify-content: center;
      align-items: center;
      border: 1px solid black;
    }

![Flex item img height has to be set to auto else the height property will override the stretch property.](https://cdn-images-1.medium.com/max/2000/1*bKsKgNEN37PyxMnw9DNEMw.png)

Flex item img 的 height 必须设置为 auto，否则 height 属性会覆盖 stretch 属性

    .container {
       height: 50vh;
      display: flex;
      flex-wrap: wrap;
      justify-content: center;
      align-items: stretch;
    border: 1px solid black;
    }

## **Flexbox Examples**

    <div class="wrapper">
      <div class="card card-1">1</div>
      <div class="card card-2">2</div>
      <div class="card card-3">3</div>
      <div class="card card-4">4</div>
    </div>
    
    .wrapper {
      height: 100%;
      display: flex;
      flex-wrap: wrap;
      justify-content: flex-start;
      align-items: center;
      border: 1px solid black;
    } 
    .card {
      display: flex;
      justify-content: center;
      align-items: center;
      width: 120px;
      height: 120px;
      background-color: green;
      margin: 10px;
      color: #fff;
      font-size: 36px;
      font-weight: 600;
    }
    .card-1 {
      background-color: red;
    }
    .card-2 {
      background-color: brown;
    }
    .card-3 {
      background-color: purple;
    }
    .card-4 {
      background-color: green;
    }

[codepen](https://codepen.io/jidelambo/pen/MOmmwV)

display: flex;
 flex-wrap: wrap;
 justify-content: flex-start;
 align-items: center;

2)

    .wrapper {
      height: 100%;
      display: flex;
      flex-wrap: wrap;
      justify-content: space-between;
      align-items: center;
      border: 1px solid black;
    }  
    .card {
      display: flex;
      justify-content: center;
      align-items: center;
      width: 120px;
      height: 120px;
      background-color: green;
      margin: 10px;
      color: #fff;
      font-size: 36px;
      font-weight: 600;
    }
    .card-1 {
      background-color: red;
    }
    .card-2 {
      background-color: brown;
    }
    .card-3 {
      background-color: purple;
    }
    .card-4 {
      background-color: green;
    }

[codepen](https://codepen.io/jidelambo/pen/XzRRNy)

## **总结**

我们已经讨论了Flex Container 的 flexbox 属性及其对齐 flex item 的影响。 我希望在后面的文章中更多地介绍Flex Item 中的 Flexbox propeties。
如果这篇文章对你有帮助，请给予一些绿色的鼓掌或下面的评论。
感谢你的阅读！

