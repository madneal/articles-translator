## Understanding CSS Flexbox

https://codeburst.io/understanding-css-flexbox-d6162885fefe

![](https://cdn-images-1.medium.com/max/2000/1*xpjy9vmoYYR_5RGPBqQoIw.png)

Have you been wondering how best to arrange elements with flexbox? If you cant stand CSS Float hacks, then lets dive in and learn how to work with Flexbox.

 <iframe src="https://medium.com/media/ed658fac807f1b8b44792a653be70dfa" frameborder=0></iframe>

## What is Flexbox?

Let’s take note of two important elements that works with Flexbox properties; Flex Container and Flex Item.

Flex Container is obviously set to be the parent element while the Flex Items are the Flex container’s direct children element.

Therefore, Flexbox layout gives the Flex container the ability to adjust it’s Flex Items’ width and height to accommodate available space for all kinds of display devices and screen sizes.
>  With the above being said, there are one or more ***flex items*** inside a ***flex container***.

## **Flexbox properties on Flex Container**

    1. **Display: Flex**

In the image below, we have four boxes and by default each box is a block element; that is, they ought to occupy a full-width of a line. Flexbox enables the flex properties on all the container’s direct children (i.e, the four boxes) and the four boxes are displayed inline.

![](https://cdn-images-1.medium.com/max/2000/1*hb6tOl-7bpB_xOV2XDG3GA.png)

    .container {
       ***display: flex;***
    }

2. **Flex Direction**

Often times we have tried aligning menu list items vertically or horizontally. We then have to use list items, then have the Unordered list’s css property as ***display-inline.***

But with Flex-direction on the Flex Container, it can help us specify the alignment direction of our Flex Items inside the container (ie, left to right ***Or*** right to left). The following are properties of flex-direction:

***row, row-reverse, column and column-reverse.***

![display: flex;
 ***flex-direction: column;***](https://cdn-images-1.medium.com/max/2000/1*vPkSdz6kf2yFFPLN_0oBYg.png)

    .container {
       display: flex;
       ***flex-direction: column;***    /** lays the Flex Items vertically from top to bottom **/
    }

![display: flex;
 ***flex-direction: row;***](https://cdn-images-1.medium.com/max/2000/1*hb6tOl-7bpB_xOV2XDG3GA.png)

    .container {
       display: flex;
       ***flex-direction: row;*** /** Places the Flex Items from left to right **/
    }

3. **Flex Wrap**

Lets assume we have a gallery of 12 profile pictures on our team’s page, we’re either left with options of having all 12 images in a row or have images wrap down onto other rows.

With Flex-wrap, we can make all images stay on one line or wrap onto another line easily. The following are properties of flex-wrap:

***nowrap, wrap and wrap-reverse.***

![images are not wrapped to another line and this looks terrible](https://cdn-images-1.medium.com/max/2000/1*10-Z2OdPKwYS3QDVNFfs-Q.png)

    .container {
       display: flex;
       ***flex-wrap: nowrap;*** /** default property. all profile pictures will be on one line **/
    }

![images will wrap onto new lines](https://cdn-images-1.medium.com/max/2000/1*49fmrP7OosgNa7l6RI_bVQ.png)

    .container {
        display: flex;   
        ***flex-wrap: wrap;*** /** images will wrap onto another line if necessary, from top to bottom **/
    }

4. **Justify Content**

We have the following properties of Justify-content:

***flex-start, flex-end, center, space-around and space-between.***

![](https://cdn-images-1.medium.com/max/2000/1*DT3irNO28CMlTeEOByVTJQ.png)

    .container {
       display: flex;
       ***justify-content: flex-start;***
       border: 1px solid black;
    }

![](https://cdn-images-1.medium.com/max/2000/1*VoPMWkKRRb-X7X8FfXR7eA.png)

    .container {
       display: flex;
       ***justify-content: flex-end;***
    border: 1px solid black;
    }

![](https://cdn-images-1.medium.com/max/2000/1*FZeechhQD1IVta2G_VJgLg.png)

    .container {
       display: flex;
       ***justify-content: center;***
    border: 1px solid black;
    }

![](https://cdn-images-1.medium.com/max/2000/1*5ZLXFMHJ3C250ZzlovArhw.png)

    .container {
       display: flex;
       ***justify-content: space-between;***
    border: 1px solid black;
    }

![](https://cdn-images-1.medium.com/max/2000/1*xA5jAgh2HzPNYeqXYXDF1w.png)

    .container {
       display: flex;
       ***justify-content: space-around;***
    border: 1px solid black;
    }

**5. Align Items**

Just like justify-content, ***align-items*** helps us align flex items on the cross-axis, that is vertically from top to bottom and in some cases from bottom to top.

We have the following properties of ***align-items***:

***flex-start, flex-end, center, stretch and baseline.***

![cards align on the top of the cross-axis](https://cdn-images-1.medium.com/max/2000/1*qVoeSLvHQf63vZ_cfG7N-g.png)

    .container {
      height: 50vh;
      display: flex;
      flex-wrap: wrap;
      justify-content: center;
      ***align-items: flex-start;***
      border: 1px solid black;
    }

![cards align on the end of the cross-axis](https://cdn-images-1.medium.com/max/2000/1*hD5JeOPEfPPawpixwZzgVA.png)

    .container {
      height: 50vh;
      display: flex;
      flex-wrap: wrap;
      justify-content: center;
      ***align-items: flex-end;***
      border: 1px solid black;
    }

![](https://cdn-images-1.medium.com/max/2000/1*vzpHtRSgzSMCRIzNjP7oGQ.png)

    .container {
      height: 50vh;
      display: flex;
      flex-wrap: wrap;
      justify-content: center;
      ***align-items: baseline;***
      border: 1px solid black;
    }

![cards are of different heights but they still align at the center](https://cdn-images-1.medium.com/max/2000/1*TOWFH-Duvckx2PZRAp9A5A.png)

    .container {
      height: 50vh;
      display: flex;
      flex-wrap: wrap;
      justify-content: center;
      ***align-items: center;***
      border: 1px solid black;
    }

![Flex item img height has to be set to auto else the height property will override the stretch property.](https://cdn-images-1.medium.com/max/2000/1*bKsKgNEN37PyxMnw9DNEMw.png)

    .container {
       height: 50vh;
      display: flex;
      flex-wrap: wrap;
      justify-content: center;
      ***align-items: stretch;***
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

 <iframe src="https://medium.com/media/8ba55d8178efd2c298178cd51c8b00e8" frameborder=0></iframe>

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

[codepen 实例](https://codepen.io/jidelambo/pen/XzRRNy)

## **Conclusion**

We have discussed the flexbox properties on Flex Container and the impact it has on aligning flex items. I hope to get more into the Flexbox propeties on Flex Items in subsequent article.

If this article has been helpful to you, kindly give some green clap below or drop a comment.

Thank you for reading!