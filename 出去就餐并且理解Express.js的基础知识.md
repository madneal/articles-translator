![](https://cdn-images-1.medium.com/max/11520/1*iMkFu1T52fkSnlZDlCrvkQ.jpeg)

## 出去就餐并且理解Express.js的基础知识

>原文：[Going out to eat and understanding the basics of Express.js](https://medium.freecodecamp.org/going-out-to-eat-and-understanding-the-basics-of-express-js-f034a029fb66)
>
>译者：[neal1991](https://github.com/neal1991)
>
>welcome to star my [articles-translator](https://github.com/neal1991/articles-translator/), providing you advanced articles translation. Any suggestion, please issue or contact [me](mailto:bing@stu.ecnu.edu.cn)
>
>LICENSE: [MIT](https://opensource.org/licenses/MIT)

如果你曾经去过一个坐下来就餐的餐厅，那么你可以了解 Express 的基础知识。 但是，如果你刚刚开始构建你的第一个 Node.js 后端......你可能并不会很顺利。

是的 - 如果你曾经有过 JavaScript 经验，学习 Node 肯定更容易。 但是，在构建后端时面临的挑战与在前端使用JavaScript 时所面临的挑战完全不同。

当我学习Node时，我选择了困难的方式。 我一遍又一遍地学习电子书，写作教程和视频，直到我终于明白我**为什么**要做我正在做的事情。

有一个更简单的方法。 我打算用一个餐馆的比喻来解释你的第一个应用程序的四个关键部分。 [Express.js](https://expressjs.com/) 是一个组织你的代码的流行框架，我会为任何初学者推荐它。 稍后我会进一步解释。

下面是我们将会涉及到的四个关键部分：

1. The require statements
2.  Middleware
3. Routing
4. App.listen()/ Starting the server

在这个比喻中，你是一个餐馆老板，希望雇用一个总经理 - 创建所有流程并且进行管理，这样餐厅就可以顺利运行，客户也就快乐了。

下面是接下来部分的预览：

![](https://cdn-images-1.medium.com/max/2578/1*gWVqib20b1NNzB6vrM-U6w.png)

最后，你将会理解基本 Express app 的每个部分的功能。

## 步骤1: 雇佣经理 (require statements)

在这个例子中，你是餐馆老板。 而且你需要聘请专家来管理你的新餐厅的日常运作。 你当然不是专家，你不能把它交给服务员和厨房去搞清楚。

如果你想经营一家高效安全的餐厅，你需要有人来保证你的员工以最高的效率工作。 Express 就是新的经理。

第一部分非常简单。 与其他 NPM 软件包一样，你需要使用 npm 安装 express 模块，然后使用 **require** statement 来加载模块。

![](https://cdn-images-1.medium.com/max/2596/1*VjyG-yoVn9aUYJ_cdN6RYA.png)

不像其它的许多 NPM 包，你也需要使用这行：

    const app = express();
这是因为你需要一个变量来保存你的新的 Express 应用程序。 Express 不是 Node 的默认部分。

## 步骤二: 在餐厅做决定 (middleware)

让我们在这停一下。餐厅里最常见的例程有哪些？我们脑海中立马出现了3个：

1. 给新顾客安排座位
2. 接受食物订单
3. 在用餐结束进行确认

对于每一个例程，都需要进行一系列的进程才能执行行动。比如，在你给顾客安排座位之前，你需要知道：

1. 他们是不是穿了衬衫和鞋子（以及裤子）？否则，他们不能被安排座位。
2. 如果他们想坐在吧台那里，他们是否已经有21岁（如果你在美国的话）

这不是海滩酒吧！ 同样，在你的代码中，你需要验证请求是否具有某些标准，然后才能继续。 例如，如果有人尝试登录到你的网站：

1. 他们是否具有账户？
2. 他们是否输入了正确的密码？

这是 **middleware** 概念的来源。Middleware 功能允许你对任何传入的请求采取行动，并在发回响应之前对其进行修改。

![](https://cdn-images-1.medium.com/max/2000/0*8HIzvtX-DA3C26uv.png)

在你的餐厅，你需要一系列的规则来决定你是否应该接待来访的人。 比方说，一对夫妇走进你的餐厅。 在给他们一张桌子之前，你有一条规则：他们穿着衬衫和鞋子吗？

![](https://cdn-images-1.medium.com/max/2636/1*Gqix0p7PBNJ5htTY3sT7OQ.png)

首先，你从[app.use()](http://expressjs.com/en/api.html#app.use)开始。这意味着这些仅仅是下一个路线需要应用的规则。它们不是 GET，POST，PUT或DELETE。

在第四行，你有一个匿名函数，参数 req，res 和 next。对于这个代码块的目的，你只是检查请求（req），看它是否有衬衫和鞋子。

你最后还需要使用next()函数，因为你只是在这里验证服装。之后，在路线中，你将允许客人获得实际的桌子。

在第五行和第六行，你检查他们是否有衬衫和鞋子。

而在第7-9行中，只有两者兼而有之才行。

上面的代码块缺少一个重要的事情：一个**路径**。这是请求中包含的特定字符串。而且由于缺少路径，它将在每个请求上运行。

你可以想象？当顾客进入餐馆...点了食物...要求支票...员工将被迫在他们上下看他们确定他们穿衣服！这是一个快速停业的办法。

![](https://cdn-images-1.medium.com/max/2578/1*fjZKIJYmTIxQmVMURYTW9g.png)

所以，我们在上面的例子中改变第4行。 现在，我们只会在用户请求“/ table”路径时运行这个代码。

完整的解释：

![](https://cdn-images-1.medium.com/max/2606/1*d1OPYjAlr6mUWtjtMRbk6g.png)

## 步骤3: 执行常见的例程 (路由)

让我们继续就座的例子。 到目前为止，我们只知道如何验证某人是否应该就做。 但是我们实际上并不知道如何把他们指引到桌子旁边并且就坐。

这就是**路由**的来源。路由允许我们根据**路径**编写具体的行动。 选项是 GET，POST ，PUT 和 DELETE，但现在我们将重点介绍 GET 和 POST。

在餐厅的环境下，我们需要创建一个 GET 请求，以便选择一个特定的桌子并安排客人就做。 GET 不会修改或添加新数据到你的数据库。 他们只是检索基于特定参数的信息。

在这种情况下，假设你需要创建一个程序来安排两个派对。 2号来自客户的要求。

![](https://cdn-images-1.medium.com/max/2572/1*pGvgMABGA1xzrSL9EFGQmQ.png)

好的，在我解释之前：是的，这只是在最后发送信息。 实际上还没有找到一个具体的表格来给客户安排座位。 我需要在一个数组中搜索一个打开的表格，这涉及到更多的背景故事......这超出了本教程的范围。

在第12行中，我们定义了当宾客**请求**'table'**路由**时查找表的过程。 就像上面的中间件示例一样，我们有可用的请求和响应参数。 它也有一个**参数**，金额。 在这个例子中，这是两个。

事实上，第12行函数声明之后的所有内容在技术上都是**middleware**，因为它修改了用户请求。 你会看到图中的结尾。

在第13行中，我们从请求对象的**参数**中访问聚会中的人数。 由于请求来自用户，所以没有声明任何地方，我们没有任何前端代码。 所以如果这是一个真正的应用程序，请求可能如下所示：

    req = {
      params: {
        amount: 2;
      }
    }

在第13行中，我们的 party 变量访问请求中的 params 对象的 amount 属性。

最后在第14行，我们发送响应给客户：我们正在寻找合适的桌子。

这是一次很多。 下面一张图表：

![](https://cdn-images-1.medium.com/max/3666/1*k7DkIw1cheKYBwu_AC4SAA.png)

## 步骤3.5: 让你的餐厅有效率 (路由)

现在您可以追踪从请求到响应的完整路径。 但是，随着你的应用程序的规模不断扩大，你将不希望单独为每条路由编码规则。 你会发现一些路由共享相同的规则，所以你需要找到一种方法来将一组规则应用到多个路由。

就座位而言，您可以将你的客户安置在吧台或餐桌旁。 他们有衬衫和鞋子的共同规则，但是在酒吧里坐着要求每个人都超过 21岁。

而且，在服务客户方面，你需要使用稍微不同的程序来供应开胃菜，主菜和晚餐。 但是，这三条路线也有很多共同之处。

这是路由器进来的地方。路由器让你分组你的路由，这样你就可以创建通用的规则。

![](https://cdn-images-1.medium.com/max/2394/1*6Irrxz4EmHaPgVm0JRgVLg.png)

我们需要创建 middleware 来覆盖这些情况。 现在我会将考虑到上面的就座案例，对上面的代码进行重写。

这是完整的代码片段：

![](https://cdn-images-1.medium.com/max/2598/1*Pih87WdfXU_PEXkcbsaAIw.png)

我将分别讨论每个部分。

在第 4 行，我们声明我们的路由器。

在第6行和第14行，我们现在使用seatingRouter.use（）来代替app.use（）来表示这个中间件只与seatRouter路由有关。

最后，在第21行中，我们添加了更多的中间件，以显示每个座位路由器以“/ seating”开头。 所以，如果有人要求在酒吧坐一个座位，完整的路径是'/ seating / bar'，这可能会有点不合理，因为当你在第4行创建路由器时，你可能会预期路径被定义。 这很正常！

这里是以图表的形式：

![](https://cdn-images-1.medium.com/max/2000/1*-1x9T6VvBCQihyzwqgnGIA.png)

并且当你添加一个 GET 路由的时候，  它将会转到你上面分配的路由。

![](https://cdn-images-1.medium.com/max/2604/1*EPEUF9z94mMlCYXMB6HENA.png)

## 步骤4: 开放营业 (端口)

好吧，最后一部分。 到目前为止，你已经雇佣了一位经理，在接受客户请求之前定义了要做的事情，并且确定如何处理特定的客户请求。现在，你只需确定所需位置的地址即可。

你的服务器端口有点像餐厅本身的地址。 由于你的服务器可以同时处理多种类型的餐厅（或服务器端脚本），因此你需要告知每个脚本应在哪里运行。

![](https://cdn-images-1.medium.com/max/2650/1*xGoTkrNMLnwyh7zR2wbbVA.png)

在上面的例子中，端口是 3000，它位于你的计算机上。 所以如果你输入：

​	`https://localhost:3000/`

在你的浏览器中，并且你在运行你的 Node 应用程序，服务器知道运行特定的脚本。 在这种情况下，只要输入URL，您将在控制台中记录消息，并能够使用任何路由。 如果餐厅本身是您的电子商务应用程序，那么它现在在地址 3000 处开始营业。

![](https://cdn-images-1.medium.com/max/2650/1*kl9doAbsvsaQNJdFWhc2-Q.png)

你喜欢这个教程吗？ 给它点赞，让别人也可以发现它。
