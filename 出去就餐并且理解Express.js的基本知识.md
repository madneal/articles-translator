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

这是 **middleware** 概念的来源。Middleware 功能允许你对任何传入的请求采取行动，并在发回响应之前对其进行修改。

![](https://cdn-images-1.medium.com/max/2000/0*8HIzvtX-DA3C26uv.png)

In your restaurant, you need a series of rules to decide if you should seat incoming people or not. Let’s say a couple walks through your door. You have one rule before giving them a table: are they wearing a shirt and shoes?

在你的餐厅，你需要一系列的规则来决定你是否应该接待来访的人。 比方说，一对夫妇走进你的餐厅。 在给他们一张桌子之前，你有一条规则：他们穿着衬衫和鞋子吗？

![](https://cdn-images-1.medium.com/max/2636/1*Gqix0p7PBNJ5htTY3sT7OQ.png)

First, you start with [app.use()](http://expressjs.com/en/api.html#app.use). This means that these are simply rules that need to be applied for the routes coming up next. They are not a GET, POST, PUT or DELETE.

In line 4, you have an anonymous function with the parameters req, res and next. For the purposes of this code block, you are just inspecting the request (req) to see if it has shirt and shoes.

You also need to use the next() function at the end because you are simply validating clothing here. Later, in the routes, you will allow the guests to get an actual table.

In lines 5 and 6, you check if they have a shirt and shoes.

And in lines 7–9, you only proceed if they have both.

The code block above is missing one important thing: A **path**. This is the specific string included with the request. And since it is missing a path, it will run on every single request.

Can you imagine? When customers entered the restaurant… ordered food… asked for the check… employees would be forced to look up and down at them to make sure they were clothed! That is a quick way to go out of business.

首先，你从[app.use()](http://expressjs.com/en/api.html#app.use)开始。这意味着这些仅仅是下一个路线需要应用的规则。它们不是 GET，POST，PUT或DELETE。

在第四行，你有一个匿名函数，参数 req，res 和 next。对于这个代码块的目的，你只是检查请求（req），看它是否有衬衫和鞋子。

你最后还需要使用next()函数，因为你只是在这里验证服装。之后，在路线中，你将允许客人获得实际的桌子。

在第五行和第六行，你检查他们是否有衬衫和鞋子。

而在第7-9行中，只有两者兼而有之才行。

上面的代码块缺少一个重要的事情：一个**路径**。这是请求中包含的特定字符串。而且由于缺少路径，它将在每个请求上运行。

你可以想象？当顾客进入餐馆...点了食物...要求支票...员工将被迫在他们上下看他们确定他们穿衣服！这是一个快速停业的办法。

![](https://cdn-images-1.medium.com/max/2578/1*fjZKIJYmTIxQmVMURYTW9g.png)

So, we change line 4 in the example above. Now, we will only run this code when a user requests along the ‘/table’ route.

The full explanation:

所以，我们在上面的例子中改变第4行。 现在，我们只会在用户请求“/ table”路径时运行这个代码。

完整的解释：

![](https://cdn-images-1.medium.com/max/2606/1*d1OPYjAlr6mUWtjtMRbk6g.png)

## 步骤三: 执行常见的例程 (路由)

Let’s continue with the seating example. So far, we only know how to validate whether someone should be seated or not. But we do not actually know how to lead them to a table and sit them down.

This is where **routes** come in. Routes allow us to script specific actions based on the **path**. The options are GET, POST, PUT and DELETE, but we will focus on GET and POST for now.

In the context of a restaurant, we need to create a GET request in order to choose a specific table and seat the guests. GETs do not modify or add to your database. They just retrieve information based on specific parameters.

In this case, let’s say that you need to create a procedure to seat a party of two. The number 2 came from the customer **request.**

让我们继续就座的例子。 到目前为止，我们只知道如何验证某人是否应该就做。 但是我们实际上并不知道如何把他们指引到桌子旁边并且就坐。

这就是**路由**的来源。路由允许我们根据**路径**编写具体的行动。 选项是 GET，POST ，PUT 和 DELETE，但现在我们将重点介绍 GET 和 POST。

在餐厅的环境下，我们需要创建一个 GET 请求，以便选择一个特定的桌子并安排客人就做。 GET 不会修改或添加新数据到你的数据库。 他们只是检索基于特定参数的信息。

在这种情况下，假设你需要创建一个程序来安排两个派对。 2号来自客户的要求。

![](https://cdn-images-1.medium.com/max/2572/1*pGvgMABGA1xzrSL9EFGQmQ.png)

Okay, before I explain: Yes, this is only sending a message at the end. It has not actually found a specific table to seat the customer yet. I would need to search an array for an open table, have more of a back story…that is outside of the scope of this tutorial.

In line 12, we define the procedure for finding a table when a guest **requests** along the ‘/table’ **route**. Just like the middleware example above, we have request and response parameters available. It also has a **parameter**, amount. This is two, in this example.

In fact, everything after the function declaration in line 12 is technically **middleware** since it modifies a user request. You will see in the diagram at the end.

In line 13, we access the number of people in the party from the **parameters** of the request object. That is not declared anywhere since the request came from the user, and we do not have any front-end code. So here is what the request might look like if this was a real app:

好的，在我解释之前：是的，这只是在最后发送信息。 实际上还没有找到一个具体的表格来给客户安排座位。 我需要在一个数组中搜索一个打开的表格，这涉及到更多的背景故事......这超出了本教程的范围。

在第12行中，我们定义了当宾客**请求**'table'**路由**时查找表的过程。 就像上面的中间件示例一样，我们有可用的请求和响应参数。 它也有一个**参数**，金额。 在这个例子中，这是两个。

事实上，第12行函数声明之后的所有内容在技术上都是**middleware**，因为它修改了用户请求。 你会看到图中的结尾。

在第13行中，我们从请求对象的**参数**中访问聚会中的人数。 由于请求来自用户，所以没有声明任何地方，我们没有任何前端代码。 所以如果这是一个真正的应用程序，请求可能如下所示：

    req = {
      params: {
        amount: 2;
      }
    }

In line 13, our party variable accesses the amount **property** of the params **object** within the **request**.

Finally in line 14, we send a **response** back to the customer: we are looking for the appropriately sized table.

That is a lot at once. Here is a diagram:

在第13行中，我们的 party 变量访问请求中的 params 对象的 amount 属性。

最后在第14行，我们发送响应给客户：我们正在寻找合适的桌子。

这是一次很多。 下面一张图表：

![](https://cdn-images-1.medium.com/max/3666/1*k7DkIw1cheKYBwu_AC4SAA.png)

## 步骤3.5: 让你的餐厅有效率 (路由)

Now you can trace the full path from request to response. But as your app grows in size, you will not want to code the rules for each route individually. You will find that some routes share the same rules, so you need to find a way to apply one set of rules to multiple routes.

In terms of seating, you can either seat your customers at the bar or at a table. Those have rules in common like shirt + shoes, but seating at the bar requires every member of the party to be 21.

And, in terms of serving customers, you will need to use a slightly different procedure for serving the appetizer, main course, and dinner. But, those three routes have plenty in common as well.

This is where the **router** comes in. The router lets you group your routes so that you can create common rules.

现在您可以追踪从请求到响应的完整路径。 但是，随着你的应用程序的规模不断扩大，你将不希望单独为每条路由编码规则。 你会发现一些路由共享相同的规则，所以你需要找到一种方法来将一组规则应用到多个路由。

就座位而言，您可以将你的客户安置在吧台或餐桌旁。 他们有衬衫和鞋子的共同规则，但是在酒吧里坐着要求每个人都超过 21岁。

而且，在服务客户方面，你需要使用稍微不同的程序来供应开胃菜，主菜和晚餐。 但是，这三条路线也有很多共同之处。

这是路由器进来的地方。路由器让你分组你的路由，这样你就可以创建通用的规则。

![](https://cdn-images-1.medium.com/max/2394/1*6Irrxz4EmHaPgVm0JRgVLg.png)

We need to create middleware to cover each of these cases. I will just cover the seating cases for now since it will overwrite the code above.

Here is the full code snippet:

我们需要创建 middleware 来覆盖这些情况。 现在我会将考虑到上面的就座案例，对上面的代码进行重写。

这是完整的代码片段：

![](https://cdn-images-1.medium.com/max/2598/1*Pih87WdfXU_PEXkcbsaAIw.png)

I am going to cover each part individually.

In line 4, we declare our router.

In lines 6 and 14, we now have seatingRouter.use() in place of app.use() to indicate that this **middleware** is only related to seatingRouter routes.

Finally, in line 21, we add more middleware to show that every seatingRouter route begins with ‘/seating’. So, if someone requested a seat at the bar, the full path would be ‘/seating/bar.’ This may feel a little out of order, since you might expect the path to be defined when you create the router in line 4. That is normal!

Here is that in diagram form:

我将分别讨论每个部分。

在第 4 行，我们声明我们的路由器。

在第6行和第14行，我们现在使用seatingRouter.use（）来代替app.use（）来表示这个中间件只与seatRouter路由有关。

最后，在第21行中，我们添加了更多的中间件，以显示每个座位路由器以“/ seating”开头。 所以，如果有人要求在酒吧坐一个座位，完整的路径是'/ seating / bar'，这可能会有点不合理，因为当你在第4行创建路由器时，你可能会预期路径被定义。 这很正常！

这里是以图表的形式：

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
