![](https://cdn-images-1.medium.com/max/11520/1*iMkFu1T52fkSnlZDlCrvkQ.jpeg)

## Going out to eat and understanding the basics of Express.js出去吃饭并且理解Express.js的基本知识

If you have ever visited a sit-down restaurant, then you can understand the basics of Express. But if you are just starting to build your first Node.js back end…you might be in for a bumpy ride.

Yes — it is certainly easier to learn Node if you have past experience with JavaScript. But the challenges you will face while building a back end are completely different than the ones you face while using JavaScript on the front end.

如果你曾经去过一个坐下来就餐的餐厅，那么你可以了解 Express 的基础知识。 但是，如果你刚刚开始构建你的第一个 Node.js 后端......你可能并不会很顺利。

是的 - 如果你曾经有过 JavaScript 经验，学习 Node 肯定更容易。 但是，在构建后端时面临的挑战与在前端使用JavaScript 时所面临的挑战完全不同。

When I learned Node, I chose the hard way. I studied eBooks, written tutorials, and videos over and over until I finally understood **why **I was doing what I was doing.

There is an easier way. I am going to use a restaurant analogy to explain four key parts of your first Express app. [Express.js](https://expressjs.com/) is a popular framework for organizing your code, and I would recommend it for any beginner. I’ll explain further in a moment.

当我学习Node时，我选择了困难的方式。 我一遍又一遍地学习电子书，写作教程和视频，直到我终于明白我**为什么**要做我正在做的事情。

有一个更简单的方法。 我打算用一个餐馆的比喻来解释你的第一个应用程序的四个关键部分。 [Express.js](https://expressjs.com/) 是一个组织你的代码的流行框架，我会为任何初学者推荐它。 稍后我会进一步解释。

Here are the four key parts we will cover:

下面是我们将会涉及到的四个关键部分：

  1. The require statements

  2. Middleware

  3. Routing

  4. App.listen()/ Starting the server

In this analogy, you are a restaurant owner looking to hire a general manager — the person who creates all the processes and manages the place so that it runs smoothly and customers leave happy.

在这个比喻中，你是一个餐馆老板，希望雇用一个总经理 - 创建所有流程并且进行管理，这样餐厅就可以顺利运行，客户也就快乐了。

Here is a preview of what is next:

下面是接下来部分的预览：

![](https://cdn-images-1.medium.com/max/2578/1*gWVqib20b1NNzB6vrM-U6w.png)

By the end, you will understand the functionality of every part of a basic Express app.

最后，你将会理解基本 Express app 的每个部分的功能。

## 步骤一: 雇佣经理 (require statements)

In this example, you are the restaurant owner. And you need to hire an expert to run the day-to-day operations of your new restaurant. You certainly aren’t an expert, and you can’t leave it to the waitstaff and kitchen to figure out.

If you want to run an efficient and safe restaurant, you need someone to keep your staff working at maximum efficiency. Express is the new manager.

在这个例子中，你是餐馆老板。 而且你需要聘请专家来管理你的新餐厅的日常运作。 你当然不是专家，你不能把它交给服务员和厨房去搞清楚。

如果你想经营一家高效安全的餐厅，你需要有人来保证你的员工以最高的效率工作。 Express 就是新的经理。

The first part is pretty straightforward. Like with any other NPM package, you need to npm install the express module and then use a **require** statement to load the module.

第一部分非常简单。 与其他 NPM 软件包一样，你需要使用 npm 安装 express 模块，然后使用 **require** statement 来加载模块。

![](https://cdn-images-1.medium.com/max/2596/1*VjyG-yoVn9aUYJ_cdN6RYA.png)

Unlike many other NPM packages, you also need to use this line:

不像其它的许多 NPM 包，你也需要使用这行：

    const app = express();
This is because you need a variable to hold your new Express application. Express is not a default part of Node.

这是因为你需要一个变量来保存你的新的 Express 应用程序。 Express 不是 Node 的默认部分。

## 步骤二: 在餐厅做决定 (middleware)

Let’s take a step back here. What are some common routines that happen at restaurants? There are three that immediately jump into my head:
让我们在这停一下。餐厅里最常见的例程有哪些？我们脑海中立马出现了3个：

  1. 给新顾客安排座位

  2. 接受食物订单

  3. 在用餐结束进行确认

For each one, there are a series of checks that you need to run before you can execute the action. For example, before you seat customers you need to know:

对于每一个例程，都需要进行一系列的进程才能执行行动。比如，在你给顾客安排座位之前，你需要知道：

  1. Are they wearing a shirt and shoes (and pants)? Otherwise, they cannot be seated.他们是不是穿了衬衫和鞋子（以及裤子）？否则，他们不能被安排座位。

  2. If they want to sit at the bar, are they 21 years old (if you are in the United States)?如果他们想坐在吧台那里，他们是否已经有21岁（如果你在美国的话）

This ain’t a beach bar! Similarly, in your code, you will need to validate that requests have certain criteria before they can continue. For example, if a person tries to log in to your site:
这不是海滩酒吧！ 同样，在你的代码中，你需要验证请求是否具有某些标准，然后才能继续。 例如，如果有人尝试登录到您的网站：

  1. Do they have an account?他们是否具有账户？

  2. Did they enter the correct password?他们是否输入了正确的密码？

This is where the concept of **middleware** comes in. Middleware functions allow you to take action on any incoming request and modify it before sending back a response.

![](https://cdn-images-1.medium.com/max/2000/0*8HIzvtX-DA3C26uv.png)

In your restaurant, you need a series of rules to decide if you should seat incoming people or not. Let’s say a couple walks through your door. You have one rule before giving them a table: are they wearing a shirt and shoes?

![](https://cdn-images-1.medium.com/max/2636/1*Gqix0p7PBNJ5htTY3sT7OQ.png)

First, you start with [app.use()](http://expressjs.com/en/api.html#app.use). This means that these are simply rules that need to be applied for the routes coming up next. They are not a GET, POST, PUT or DELETE.

In line 4, you have an anonymous function with the parameters req, res and next. For the purposes of this code block, you are just inspecting the request (req) to see if it has shirt and shoes.

You also need to use the next() function at the end because you are simply validating clothing here. Later, in the routes, you will allow the guests to get an actual table.

In lines 5 and 6, you check if they have a shirt and shoes.

And in lines 7–9, you only proceed if they have both.

The code block above is missing one important thing: A **path**. This is the specific string included with the request. And since it is missing a path, it will run on every single request.

Can you imagine? When customers entered the restaurant… ordered food… asked for the check… employees would be forced to look up and down at them to make sure they were clothed! That is a quick way to go out of business.

![](https://cdn-images-1.medium.com/max/2578/1*fjZKIJYmTIxQmVMURYTW9g.png)

So, we change line 4 in the example above. Now, we will only run this code when a user requests along the ‘/table’ route.

The full explanation:

![](https://cdn-images-1.medium.com/max/2606/1*d1OPYjAlr6mUWtjtMRbk6g.png)

## Step 3: executing common routines (routing)

Let’s continue with the seating example. So far, we only know how to validate whether someone should be seated or not. But we do not actually know how to lead them to a table and sit them down.

This is where **routes** come in. Routes allow us to script specific actions based on the **path**. The options are GET, POST, PUT and DELETE, but we will focus on GET and POST for now.

In the context of a restaurant, we need to create a GET request in order to choose a specific table and seat the guests. GETs do not modify or add to your database. They just retrieve information based on specific parameters.

In this case, let’s say that you need to create a procedure to seat a party of two. The number 2 came from the customer **request.**

![](https://cdn-images-1.medium.com/max/2572/1*pGvgMABGA1xzrSL9EFGQmQ.png)

Okay, before I explain: Yes, this is only sending a message at the end. It has not actually found a specific table to seat the customer yet. I would need to search an array for an open table, have more of a back story…that is outside of the scope of this tutorial.

In line 12, we define the procedure for finding a table when a guest **requests** along the ‘/table’ **route**. Just like the middleware example above, we have request and response parameters available. It also has a **parameter**, amount. This is two, in this example.

In fact, everything after the function declaration in line 12 is technically **middleware** since it modifies a user request. You will see in the diagram at the end.

In line 13, we access the number of people in the party from the **parameters** of the request object. That is not declared anywhere since the request came from the user, and we do not have any front-end code. So here is what the request might look like if this was a real app:

    req = {
      params: {
        amount: 2;
      }
    }

In line 13, our party variable accesses the amount **property** of the params **object** within the **request**.

Finally in line 14, we send a **response** back to the customer: we are looking for the appropriately sized table.

That is a lot at once. Here is a diagram:

![](https://cdn-images-1.medium.com/max/3666/1*k7DkIw1cheKYBwu_AC4SAA.png)

## Step 3.5: making your restaurant efficient (router)

Now you can trace the full path from request to response. But as your app grows in size, you will not want to code the rules for each route individually. You will find that some routes share the same rules, so you need to find a way to apply one set of rules to multiple routes.

In terms of seating, you can either seat your customers at the bar or at a table. Those have rules in common like shirt + shoes, but seating at the bar requires every member of the party to be 21.

And, in terms of serving customers, you will need to use a slightly different procedure for serving the appetizer, main course, and dinner. But, those three routes have plenty in common as well.

This is where the **router** comes in. The router lets you group your routes so that you can create common rules.

![](https://cdn-images-1.medium.com/max/2394/1*6Irrxz4EmHaPgVm0JRgVLg.png)

We need to create middleware to cover each of these cases. I will just cover the seating cases for now since it will overwrite the code above.

Here is the full code snippet:

![](https://cdn-images-1.medium.com/max/2598/1*Pih87WdfXU_PEXkcbsaAIw.png)

I am going to cover each part individually.

In line 4, we declare our router.

In lines 6 and 14, we now have seatingRouter.use() in place of app.use() to indicate that this **middleware** is only related to seatingRouter routes.

Finally, in line 21, we add more middleware to show that every seatingRouter route begins with ‘/seating’. So, if someone requested a seat at the bar, the full path would be ‘/seating/bar.’ This may feel a little out of order, since you might expect the path to be defined when you create the router in line 4. That is normal!

Here is that in diagram form:

![](https://cdn-images-1.medium.com/max/2000/1*-1x9T6VvBCQihyzwqgnGIA.png)

And, when you add a GET route, it goes above the last statement where you assign routes to the router.

![](https://cdn-images-1.medium.com/max/2604/1*EPEUF9z94mMlCYXMB6HENA.png)

## Step 4: opening for business (ports)

Okay, last part. So far, you have hired a manager, defined what to do before accepting customer requests, and determined what to do with specific customer requests once they come in. Now, you just need to determine the address for the location where all this will happen.

Your server has **ports **that are kind of like the address for the restaurant itself. ****Since your server can handle many types of restaurants (or server-side scripts) at once, you need to tell it where each script should run.

![](https://cdn-images-1.medium.com/max/2650/1*xGoTkrNMLnwyh7zR2wbbVA.png)

In the example above, the port is 3000 and it is located on your computer. So if you type:

    [https://localhost:3000/](https://localhost:3000/)

into your browser, and you are running your Node app, the server knows to run the specific script. In this case, as soon as you enter the URL, you will log the message in the console and be able to use any of your **routes**. If the restaurant itself is your entire app, then it is now open for business at the address 3000.

![](https://cdn-images-1.medium.com/max/2650/1*kl9doAbsvsaQNJdFWhc2-Q.png)

Did you enjoy this? Give it a clap so others can discover it as well. And, if you want to get notified when I release future tutorials that use analogies, sign up here:

 <iframe src="https://medium.com/media/614143bb18105e1285ae4a1df769c191" frameborder=0></iframe>
