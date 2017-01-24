# JavaScript中继承常见的误区

[原文](https://medium.com/javascript-scene/common-misconceptions-about-inheritance-in-javascript-d5d9bab29b0a#.uok3y2bla)

[neal1991](https://github.com/neal1991)

**WAT? [**wat**] — **interjection:A sound a programmer makes when something violates the principle of least astonishment by astonishing them with counter-intuitive behavior.

```javascript
> .1 + .2
0.30000000000000004
> WAT? OMG! STFU! STUPID JAVASCRIPT!!!
…
```

Also, WAT? is the sound I make when I talk to many seasoned JavaScript developers who have neglected to learn the basic mechanics of prototypal inheritance: one of the most important innovations in CS history, and one of the [Two Pillars of JavaScript](https://medium.com/javascript-scene/the-two-pillars-of-javascript-ee6f3281e7f3).

To me, this is like a professional photographer who has yet to learn the exposure triangle — the basic formula for controlling much of the visual style of a photograph. Put simply:

If you don’t understand prototypes,
you don’t understand JavaScript.

### Aren’t classical inheritance and prototypal inheritance really the same thing, just a stylistic preference?

**No.**

Classical and prototypal inheritance are **fundamentally and semantically distinct.**

There are some **defining characteristics** between classical inheritance and prototypal inheritance. For any of this article to make sense, you must keep these points in mind:

In **class inheritance, instances inherit from a blueprint** (the class), and **create sub-class relationships**. In other words, you can’t use the class like you would use an instance. **You can’t invoke instance methods on a class definition itself.** You must first create an instance and then invoke methods on that instance.

In prototypal inheritance, **instances inherit from other instances. **Using **delegate prototypes** (setting the prototype of one instance to refer to an **examplar object**), it’s literally **Objects Linking to Other Objects**, or **OLOO**, as Kyle Simpson calls it. Using **concatenative inheritance**, you just **copy properties** from an **exemplar object** to a new instance.

It’s really important that you understand these differences. Class inheritance by virtue of its mechanisms **create class hierarchies as a side-effect of sub-class creation.** Those hierarchies lead to **arthritic code** (hard to change) and **brittleness** (easy to break due to rippling side-effects when you modify base classes).

Prototypal inheritance does not necessarily create similar hierarchies. I recommend that you keep prototype chains as shallow as possible. It’s easy to flatten many prototypes together to form a **single delegate prototype.**

TL;DR:

- A **class **is a **blueprint**.
- A **prototype** is an **object** instance.

### Aren’t classes the right way to create objects in JavaScript?

> **No.**

There are several **right ways** to create objects in JavaScript. The first and most common is an object literal. It looks like this (in ES6):

Of course, object literals have been around a lot longer than ES6, but they lack the method shortcut seen above, and you have to use *`var`* instead of *`let`*. Oh, and that template string thing in the *`.describe()`* method won’t work in ES5, either.

You can attach delegate prototypes with *`Object.create()`* (an ES5 feature):

Let’s break this one down a little. *`animal`* is a **delegate prototype**. *`mouse`* is an instance. When you try to access a property on *`mouse` *that isn’t there, the JavaScript runtime will look for the property on *`animal`* (the delegate).

**`Object.assign()`** is a new ES6 feature championed by Rick Waldron that was previously implemented in a few dozen libraries. You might know it as *`$.extend()`* from jQuery or *`_.extend()`* from Underscore. Lodash has a version of it called *`assign()`. *You pass in a destination object, and as many source objects as you like, separated by commas. It will copy all of the **enumerable own properties** by *assignment* from the source objects to the destination objects with **last in priority. **If there are any property name conflicts, the version from the last object passed in wins.

**`Object.create()`** is an ES5 feature that was championed by Douglas Crockford so that we could attach delegate prototypes without using constructors and the *`new`* keyword.

I’m skipping the constructor function example because I can’t recommend them. I’ve seen them abused a lot, and I’ve seen them cause [a lot of trouble](https://medium.com/javascript-scene/how-to-fix-the-es6-class-keyword-2d42bb3f4caf). It’s worth noting that a lot of smart people disagree with me. Smart people will do whatever they want.

Wise people will **take Douglas Crockford’s advice:**

> “If a feature is sometimes dangerous, and there is a better option, then always use the better option.”

### Don’t you need a constructor function to specify object instantiation behavior and handle object initialization?

> **No.**

Any function can create and return objects. When it’s not a constructor function, it’s called a **factory function.**

#### The Better Option

I usually don’t name my factories “factory” — that’s just for illustration. Normally I just would have called it *`mouse()`.*

### Don’t you need constructor functions for privacy in JavaScript?

> **No.**

In JavaScript, any time you export a function, that function has access to the outer function’s variables. When you use them, the JS engine creates a **closure**. Closures are a common pattern in JavaScript, and they’re commonly used for data privacy.

Closures are not unique to constructor functions. Any function can create a closure for data privacy:

![img](https://cdn-images-1.medium.com/max/1000/1*PZMdDrJu--I-WSKbrJL7Nw.jpeg)

### Does `new` mean that code is using classical inheritance?

> **No.**

The `*new`* keyword is used to invoke a constructor. What it actually does is:

- Create a new instance
- Bind `*this`* to the new instance
- Reference the new object’s delegate [[Prototype]] to the object referenced by the constructor function’s *`prototype`* property.
- Names the object type after the constructor, which you’ll notice mostly in the debugging console. You’ll see `*[Object Foo]`,* for example, instead of `*[Object object]`.*
- Allows `*instanceof`* to check whether or not an object’s prototype reference is the same object referenced by the *.prototype* property of the constructor.

#### *`instanceof`* lies

Let’s pause here for a moment and reconsider the value of *`instanceof`*. You might change your mind about its usefulness.

**Important:** `*instanceof`* does not do type checking the way that you expect similar checks to do in strongly typed languages. Instead, it does an identity check on the prototype object, and it’s easily fooled. It won’t work across execution contexts, for instance (a common source of bugs, frustration, and unnecessary limitations). For reference, an [example in the wild, from bacon.js](https://github.com/baconjs/bacon.js/issues/296).

It’s also easily tricked into false positives (and more commonly) false negatives from another source. Since it’s an identity check against a target object’s `*.prototype`* property, it can lead to strange things:

```
> function foo() {}
> var bar = { a: ‘a’};
> foo.prototype = bar; // Object {a: “a”}
> baz = Object.create(bar); // Object {a: “a”}
> baz instanceof foo // true. oops.
```

That last result is completely in line with the JavaScript specification. Nothing is broken — it’s just that *`instanceof`* can’t make any guarantees about type safety. **It’s easily tricked** into reporting both **false positives**, and **false negatives**.

Besides that, trying to force your JS code to behave like strongly typed code can block your functions from being lifted to generics, which are much more reusable and useful.

**`instanceof` limits the reusability of your code, and potentially introduces bugs into the programs that use your code.**

------

> `instanceof` lies.

> Ducktype instead.

------

#### `new` is weird

**WAT?** `*new*` also does some *weird stuff* to return values. If you try to return a primitive, it won’t work. If you return any other arbitrary object, that **does** work, but *`this`* gets thrown away, breaking all references to it (including `*.call()`* and `*.apply()`*), and breaking the link to the constructor’s *`.prototype`* reference.

### Is There a Big Performance Difference Between Classical and Prototypal Inheritance?

> **No.**

You may have heard of **hidden classes, **and think that constructors dramatically outperform objects instantiated with *`Object.create()`*. Those performance differences are **dramatically overstated.**

A small fraction of your application’s time is spent running JavaScript, and a miniscule fraction of that time is spent accessing properties on objects. In fact, the slowest laptops being produced today can access *millions of properties per second.*

**That’s not your app’s bottleneck**. Do yourself a favor and [profile your app](http://www.paulirish.com/2015/advanced-performance-audits-with-devtools/)to **discover your real performance bottlenecks**. I’m sure there are a million things you should fix before you spend another moment thinking about micro-optimizations.

**Not convinced?** For a micro-optimization to have any appreciable impact on your app, you’d have to loop over the operation **hundreds of thousands of times**, and the only differences in micro-optimization you should ever be concerned about are the ones that are **orders of magnitude apart**.

**Rule of thumb:** Profile your app and eliminate as many loading, networking, file I/O, and rendering bottlenecks as you can find. **Then and only then should you start to think about a micro-optimization.**

Can you tell the difference between *.0000000001* seconds and *.000000001* seconds? Neither can I, but I sure can tell the difference between loading 10 small icons or loading one web font, instead!

If you do **profile your app** and find that object creation really is a bottleneck, the fastest way to do it is not by using *`new`* and classical OO. **The fastest way is to use object literals**. You can do so in-line with a loop and add objects to an object pool to avoid thrashing from the garbage collector. If it’s worth abandoning prototypal OO over perf, it’s worth ditching the prototype chain and inheritance altogether to crank out object literals.

> But Google said class is fast…

**WAT? **Google is building a JavaScript engine. You are building an application. Obviously what they care about and what you care about should be **very different things**. Let Google handle the micro-optimizations. You worry about **your app’s real bottlenecks**. I promise, you’ll get a whole lot better ROI focusing on just about anything else.

### Is There a Big Difference in Memory Consumption Between Classical and Prototypal?

> **No.**

Both can use delegate prototypes to share methods between many object instances. Both can use or avoid wrapping a bunch of state into closures.

In fact, if you start with factory functions, it’s easier to switch to object pools so that you can manage memory more carefully and avoid being blocked periodically by the garbage collector. For more on why that’s awkward with constructors, see the **WAT?** note under *“Does `new` mean that code is using classical inheritance?”*

In other words, if you want the most flexibility for memory management, use factory functions instead of constructors and classical inheritance.

------

> “…if you want the most flexibility for memory management,
> use factory functions…”

------

### The Native APIs use Constructors. Aren’t they More Idiomatic than Factories?

> **No.**

**Factories are extremely common in JavaScript.** For instance, the most popular JavaScript library of all time, **jQuery** exposes a factory to users. John Resig has written about the choice to use a factory and prototype extension rather than a class. Basically, it boils down to the fact that he didn’t want callers to have to type *`new`* every time they made a selection. What would that have looked like?

What else exposes factories?

- **React** *`React.createClass()`* is a factory.
- **Angular** uses classes & factories, but wraps them all with a factory in the Dependency Injection container. All providers are sugar that use the *`.provider()`* factory. There’s even a `.*factory()`* provider, and even the *`.service()`* provider wraps normal constructors and exposes … you guessed it: A **factory** for DI consumers.
- **Ember** *`Ember.Application.create();`* is a factory that produces the app. Rather than creating constructors to call with *`new`*, the *`.extend()`* methods augment the app.
- **Node** core services like *`http.createServer()`* and *`net.createServer()` *are factory functions.
- **Express** is a factory that creates an express app.

As you can see, virtually all of the most popular libraries and frameworks for JavaScript make heavy use of factory functions. **The only object instantiation pattern more common than factories in JS is the object literal.**

JavaScript built-ins started out using constructors because Brendan Eich was told to make it look like Java. JavaScript continues to use constructors for self-consistency. It would be awkward to try to change everything to factories and deprecate constructors now.

------

> That doesn’t mean that **your APIs** have to suck.

------

### Isn’t Classical Inheritance More Idiomatic than Prototypal Inheritance?

> **No.**

Every time I hear this misconception I am tempted to say, **“do u even JavaScript?”** and move on… but I’ll resist the urge and set the record straight, instead.

Don’t feel bad if this is your question, too. It’s **not your fault**. [JavaScript Training Sucks!](https://medium.com/javascript-scene/javascript-training-sucks-284b53666245)

The answer to this question is a big, gigantic

## No… (but)

Prototypes are the idiomatic inheritance paradigm in JS, and *`class` *is the marauding invasive species.

#### A brief history of popular JavaScript libraries:

In the beginning, everybody wrote their own libs, and open sharing wasn’t a big thing. And then **Prototype** came along. (The name is a big hint here). Prototype did its magic by extending built-in **delegate prototypes** using **concatenative inheritance**.

Later we all realized that modifying built-in prototypes was an anti-pattern when native alternatives and conflicting libs broke the internet. But that’s a different story.

Next on the JS lib popularity roller coaster was **jQuery**. jQuery’s big claim to fame was **jQuery plugins**. They worked by extending jQuery’s **delegate prototype **using **concatenative inheritance.**

> Are you starting to sense a pattern here?

**jQuery remains the most popular JavaScript library ever made. By a HUGE margin. HUGE.**

This is where things get muddled and class extension starts to sneak into the language… John Resig (author of jQuery) wrote about *Simple Class Inheritance in JavaScript*, and people started *actually using it,* even though John Resig himself didn’t think it belonged in jQuery (because **prototypal OO did the same job better**).

Semi-popular Java-esque frameworks like ExtJS appeared, ushering in the first kinda, sorta, not-really mainstream uses of class in JavaScript. This was 2007. JavaScript was **12 years old** before a somewhat popular lib started exposing JS users to classical inheritance.

Three years later, Backbone **exploded** and had an *`.extend()`* method that mimicked class inheritance, including all its nastiest features such as brittle object hierarchies. That’s when *all hell broke loose.*

> ~100kloc app starts using Backbone. A few months in I’m debugging a 6-level hierarchy trying to find a bug. Stepped through every line of constructor code up the `super` chain. Found and fixed the bug in the top level base class. Then had to fix a lot of child classes because they depended on the buggy behavior of the base class. **Hours of frustration** that **should have been a 5 minute fix**.

**This is not JavaScript.** I was suddenly living in *Java hell* again. That lonely, dark, scary place where any quick movements could cause entire hierarchies to shudder and collapse in coalescing, tight-coupled convulsions.

**These are the monsters rewrites are made of.**

But, squirreled away in the Backbone docs, a ray of golden sunshine:

Our old friend, **concatenative inheritance** saving the day with a *`Backbone.Events`* **mixin**.

It turns out, if you look at **any non-trivial JavaScript library** closely enough, **you’re going to find examples of concatenation and delegation**. It’s so common and automatic for JavaScript developers to do these things that **they don’t even think of it as inheritance**, even though it **accomplishes the same goal.**

------

> Inheritance in JS is so easy
> it confuses people who expect it to take effort.

> To make it harder, we added `class`.

------

And how did we add class? **We built it on top of prototypal inheritance** using **delegate prototypes** and **object concatenation**, of course!

**That’s like driving your Tesla Model S to a car dealership and trading it in for a rusted out 1983 Ford Pinto.**

### Doesn’t the Choice Between Classical and Prototypal Inheritance Depend on the Use Case?

> **No.**

Prototypal OO is simpler, more flexible, and a lot less error prone. I have been making this claim and challenging people to come up with a compelling class use case *for many years. *Hundreds of thousands of people have heard the call. The few answers I’ve received depended on one or more of the misconceptions addressed in this article.

I was once a classical inheritance fan. I bought into it completely. I built object hierarchies everywhere. I built visual OO Rapid Application Development tools to help software architects design object hierarchies and relationships that made sense. It took a visual tool to truly map and graph the object relationships in enterprise applications using classical inheritance taxonomies.

Soon after my transition from C++ and Java to JavaScript, I stopped doing all of that. Not because I was building less complex apps (the opposite is true), but because JavaScript was so much simpler, I had no more need for all that OO design tooling.

I used to do application design consulting and frequently recommend sweeping rewrites. Why? Because **all object hierarchies are eventually wrong for new use cases.**

I wasn’t alone. In those days, **complete rewrites** were very common for new software versions. Most of those rewrites were necessitated by legacy lock-in caused by [arthritic, brittle class hierarchies](https://medium.com/javascript-scene/the-two-pillars-of-javascript-ee6f3281e7f3). Entire books were written about OO design mistakes and how to avoid them or refactor away from them. It seemed like every developer had a copy of [“Design Patterns”](http://www.amazon.com/gp/product/0201633612?ie=UTF8&camp=213733&creative=393185&creativeASIN=0201633612&linkCode=shr&tag=eejs-20&linkId=QYF6ABRMZ4O6KML2) on their desk.

I recommend that you follow the [Gang of Four’s advice](http://www.amazon.com/gp/product/0201633612?ie=UTF8&camp=213733&creative=393185&creativeASIN=0201633612&linkCode=shr&tag=eejs-20&linkId=QYF6ABRMZ4O6KML2) on this point:

> “Favor object composition over class inheritance.”

In Java, that was harder than class inheritance because you actually had to use classes to achieve it.

In JavaScript, we don’t have that excuse. It’s actually **much easier **in JavaScript to simply create the object that you need by assembling various **prototypes** together than it is to manage object hierarchies.

**WAT? **Seriously. Want the jQuery object that can turn any date input into a *`megaCalendarWidget`*? You don’t have to*`extend`* a *`class`*. JavaScript has dynamic object extension, and jQuery exposes its own prototype so you can just extend that — without an extend keyword! **WAT?:**

The next time you call the jQuery factory, you’ll get an instance that can make your date inputs mega awesome.

Similarly, you can use *`Object.assign()` *to compose any number of objects together with last-in priority:

No, really — **any number of objects:**

This technique is called **concatenative inheritance**, and the prototypes you inherit from are sometimes referred to as **exemplar prototypes**, which differ from delegate prototypes in that you **copy from them**, rather than delegate to them.

### ES6 Has the `class` keyword. Doesn’t that mean we should all be using it?

> **No.**

There are lots of [compelling reasons](https://medium.com/javascript-scene/the-two-pillars-of-javascript-ee6f3281e7f3) to [avoid the ES6 *`class`* keyword](https://medium.com/javascript-scene/how-to-fix-the-es6-class-keyword-2d42bb3f4caf), not least of which because it’s an awkward fit for JavaScript.

We already have an **amazingly powerful and expressive object system** in JavaScript. The concept of class as it’s implemented in JS today is more restrictive (in a bad way, not in a cool type-correctness way), and obscures the [very cool prototypal OO system](http://ericleads.com/2013/02/fluent-javascript-three-different-kinds-of-prototypal-oo/) that was built into the language a long time ago.

You know what would really be good for JavaScript? Better sugar and abstractions built on top of prototypes **from the perspective of a programmer familiar with prototypal OO.**

That could be [really cool](https://github.com/ericelliott/stampit).

------

> I’m creating a whole on-line class on
> Prototypal OO in JavaScript.

> [Preorder Now](https://ericelliottjs.com/product/lifetime-access-pass/)
> for Lifetime Access to all
> my JavaScript courses.

------

**Eric Elliott*** is the author of *[*“Programming JavaScript Applications”*](http://pjabook.com/)* (O’Reilly), & host of the documentary film-in-production, ***“Programming Literacy”***. He has contributed to software experiences for ***Adobe Systems***, ***Zumba Fitness***, ***The Wall Street Journal***, ***ESPN***, ***BBC***, and top recording artists including ***Usher***, ***Frank Ocean***, ***Metallica***, and many more.*

*He spends most of his time in the San Francisco Bay Area with the most beautiful woman in the world.*

