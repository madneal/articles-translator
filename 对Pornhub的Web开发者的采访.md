# Pornhub Web 开发者访谈

>原文：[Interview with a Pornhub Web Developer](https://davidwalsh.name/pornhub-interview)
>
>译者：[neal1991](https://github.com/neal1991)
>
>welcome to star my [articles-translator](https://github.com/neal1991/articles-translator/), providing you advanced articles translation. Any suggestion, please issue or contact [me](mailto:bing@stu.ecnu.edu.cn)
>
>LICENSE: [MIT](https://opensource.org/licenses/MIT)

Regardless of your stance on pornography, it would be impossible to deny the massive impact the adult website industry has had on pushing the web forward. From pushing the browser's video limits to [pushing ads through WebSocket](https://medium.com/thebugreport/pornhub-bypasses-ad-blockers-with-websockets-cedab35a8323) so ad blockers don't detect them, you have to be clever to innovate at the bleeding edge of the web.

I was recently lucky enough to interview a Web Developer at the web's largest adult website: Pornhub. I wanted to learn about the tech, how web APIs can improve, and what it's like working on adult websites. Enjoy!

*Note: The adult industry is very competitive so there were a few questions they could not answer.  I respect their need to keep their tricks close to the vest.*

**Adult sites obviously display lots of graphic content.  During the development process, are you using lots of placeholder images and videos?  How far is the development content and experience from the end product?**  

We actually don’t use placeholders when we are developing the sites! In the end, what matters is the code and functionality, the interface is something we are very used to at this point. There’s definitely a little bit of a learning curve at first, but we all got used to it pretty quickly. 

**When it comes to cam streams and third party ad scripts, how do you mock such important, dynamic resources during site and feature development?**

For development, the player is broken into two components.  The basic player implements the core functionality and fires events.  Development is done in a clean room. For integration on the sites, we want those third-party scripts and ads running so we can find problems as early in the process as possible.  For special circumstances we’ll work with advertisers to allow us to manually trigger events that might normally be random.

**An average page probably has at least one video, GIF advertisements, a few cam performer previews, and thumbnails of other videos.  How do you measure page performance and how do you keep the page as performant as possible? Any tricks you can share?**

We use a few measurement systems. 

* Our player reports metrics back to us about video playback performance and general usage
* A third-party RUM system for general site performance.
* WebpageTest private instances to script tests in the available AWS data centers.  We use this mostly for seeing what might have been going on at a given time. It also allows us to view the “waterfall” from different locations and providers.

**I have to assume the most important and complex feature on the front-end is the video player.  From incorporating ads before videos, marking highlight moments of the video, changing video speed, and other features, how do you maintain the performance, functionality, and stability of this asset?**

We have a dedicated team working strictly on the video player, their first priority is to constantly monitor for performance and efficiency. To do so we use pretty much everything that is available to us; browsers performance tools, web page tests, metrics  etc. The stability and quality is assured by a solid QA round for any updates we do. 

**How many people are on the dedicated video team?  How many front-end developers are on the team?**

I’d say given the size of the product the team size is lean to average. 

**During your time working on adult websites, how have you seen the front-end landscape change?  What new Web APIs have made your life easier?**

I’ve definitely seen a lot of improvements on every single aspect of the frontend world;

* From plain CSS to finally using LESS and Mixins, to a flexible Grid system with media queries and picture tags to accommodate different resolutions and screen sizes
* jQuery and jQueryUI are slowly moving away, so we are going back to more efficient object oriented programming in vanilla JS. The frameworks are also very interesting in some cases
* We love the new IntersectionObserver API, very useful for a more efficient way to load images
* We started playing with the Picture-in-Picture API  as well, to have that floating video on some of our pages, mainly to get user feedback about the idea.

**Looking forward, are there any Web APIs that you’d love changed, improved, or even created?**

Some of them that we would like changed or improved; Beacon, WebRTC, Service Workers and Fetch:

* Beacon: some IOS issues where it doesn’t quite work with pageHide events
* Fetch:  No download progress and doesn’t provide a way to intercept requests
* WebRTC:  Simulcast layers are limited even for screenshare, if the resolution is not big enough
* Service Workers: Making calls to navigator.serviceWorker.register isn't intercepted by any service worker's Fetch event handlers

**WebVR is has been improving in the past few years -- how useful is WebVR in its current state and how much of an effort are adult sites putting into support for VR content?  Do haptics have a role in WebVR on your sites?**

We’re investigating webXR and how to best adapt to emerging spatial computing use cases, and as the largest distribution platform we need to support creators and users however they want to experience our content. But we’re still exploring what content and platforms should be like in these new mediums.

We were the first major platform to support VR, computer vision, and virtual performers, and will continue to push new technology and the open web. 

**With so many different types of media and content on each page, what are the biggest considerations when it comes to desktop vs. mobile, if any?** 

Functionality restricted by operating system and browsers type mainly. iOS vs Android is the perfect example when it comes to a completely different set of access and features. 

For example, some iOS Mobile devices don’t allow us to have a custom video player while in Fullscreen, they force the native QuickTime player. That has to be considered when we develop new ideas. Android on the other hand gives us complete control and we can push our features to the Fullscreen mode.

Adaptive streaming in HLS is also another example, IE and Edge are picky when it comes to HLS streaming quality, in that we need to prevent certain of the higher qualities, otherwise the video would constantly stutter and have artifacts.

**What is the current minimum browser support for the adult sites you work on?  Is Internet Explorer phased out?**

We supported IE for a very long time but recently dropped support for anything older than IE11. With it we also stopped working with Flash for the video player. We are focusing on Chrome, Firefox and Safari mainly. 

**More broadly, can you share a little about the typical adult site’s stack?  Server and/or front-end? Which libraries are you using?**

Most of our sites use the following as a base:

* Nginx
* PHP
* MySQL
Memcached and/or Redis

Other technologies like Varnish, ElasticSearch, NodeJS, Go, Vertica are used where appropriate.

For frontend, we run mostly vanilla Javascript, we’re slowly getting rid of jQuery and we are just beginning to play with frameworks, mostly Vue.js

**From an outsider’s perspective, adult sites generally seem to be very much alike:  lots of video thumbnails, aggregated video content, cam performers, adverts. As someone who works on them, what are the differentiating features that make adult sites unique?**

We work very hard to give each brand some uniqueness at different levels; content library, UX and features sets, and across a lot of different algorithms. 

**Before applying and interviewing for your current employer, what were your thoughts on potentially working on adult sites?  Did you have any hesitations? If so, how were your fears to put rest?**

It never really bothered me, in the end the challenge was so appealing. The idea of millions of people potentially interacting with features I worked on was really motivating. That proved to be true very quickly, the first time something I worked on went live, I was super proud, and I indeed told all my friends to go check it out! The fact that porn will never die is reassuring for job stability as well!

**In as far as end product, sharing that you work on adult sites may not be the same as working at a local web agency.  Is there a stigma attached to telling friends, family, and acquaintances you work on adult sites? Is there any hesitance in telling people you work on adult sites?**

I’m very proud to work on these products, those close to me are aware and fascinated by it. It’s always an amazing source of conversation, jokes and is genuinely interesting. 

**Having worked at agencies outside the adult industry, is there a difference in atmosphere when working on adult sites?**

The atmosphere here is very relaxed and friendly. I don’t notice any major differences with respect to work culture at other agencies, other than the fact that it’s much bigger here than anywhere I have worked previously. 

Being a front-end developer, which teams do you work most closely with?  What are the most common daily communication methods?

We work equally with backend developers, QA testers and product managers - most of the time we simply go up to each other’s desk and talk. If not, chat (MS Teams) is very common. Then come emails.

**Lastly, is there anything you’d like to share as a front-end developer working on adult sites?**

It’s really exciting being a part of creating how users experience such a widely used product. We are generally at the forefront of trends and big changes in tech as they roll out, which keeps it fun and challenging.

*Interview end*

I found our interview really enlightening. I was a bit surprised they didn't use images while developing features and designs. It's exciting to see that Pornhub continues to push the bleeding edge of the web with WebXR, WebRTC, and Intersection Observer. I was also happy to see that they consider the current set of web APIs sufficient to start dropping jQuery.

I really wish I'd have been able to get more specific tech tips out of them; performance and clever hacks especially. I'm sure there's a wealth of knowledge to be learned behind their source code! What questions would you have asked?

