# Blazor ServerSide- Thinking about the web from a different angle

## A short history lesson
In this day and age, there are more ways to build for the web than you can count. You have multiple different approaches, languages and stacks.

Until relatively recently, the prefered approach when any kind of dynamic content was involved was to render the front end on the back end. Technologies like PHP, ASP class and ASP.Net.

This evolved over time and implemented client side features via Ajax - asyncronous javascript and xml. This allowed the clients to be more responsive, changing data on the page live rather than forcing a reload.

Due to this jscript took on a new lease of life and took us to where we are now - entire javascript frameworks that are back end agnostic - having the back end as purely an api with all front end logic running in the client meaning a much more fluid experience in return for heavier client responsibility and arguably a more difficult development experience.

Overtime, these javascript frameworks matured, starting with knockout.js, a few versions of angular and now React.

## Blazors big selling point

This move from exposing the UI from the backend in server languages like c# to exposing in the front end with javascript meant theres a large number of developers outside of their comfort zone.

Its also not hard to argue that javascript isn't ideal for larger project. It suffers from a lack of type safety and thus coersion hurdles. It can behave different in different browsers (less of an issue these days) and even has fundamental issues with basic maths. Its the best programming language we have in the browser mostly due to it being the only language we have in the browser.

That was true until WASM - Web Assembly - mozilla's genius simplification of javascript down to the bare essentials - but amazingly optimised - the idea being it can be used a compile too target - its not supposed to be used by devs - but compiler tools.

This enabled microsoft and Xamarin to produce Blazor - a new take on the client - powered by a complete version of .net (mono) running on WASM. This means your client code is any .net supported language such as C# - allowing for enterprise standards in front end code.

You still need to follow the same pattern, with the front end accessing the back end via APIs etc - but if your backend is also c#, this allows a lot of code reuse - especially around models and the like.

This is an amazing technical achievement. It's also not that useful. At this point in time, javascript is THE defacto standard for front end and thus moving away from it has a huge number of downsides.

## Blazors twin

Blazor actually comes in two flavours, client side and server side. Server side is where things get very interesting. It's code compatible with client side but works very differently, more like the frameworks of old - it runs entirely on the server, even "client" code. The difference from the frameworks of yesteryear is how this client code pushes the ui to the client - rather than full page refreshes and the like, the server manages a local DOM and sends deltas of this DOM to the client using SignalR. This means there is almost no load on the front end, the experience is very similar to a javascript framework - but most interestingly - the client never sees the client code. This opens up a number of options that let you think about things differently.

## So where do we live?

When moving code from a Blazor client side project to a server side project - the first thing you will probably notice is how fast API calls are - because realistically, they arent going across the web - both the front end and backend are running in the same place - you have direct access to the API server.

But if this is true - your front end code has the same access as the back end. You can write front end code to directly access your SQL servers, your file system etc. You probably shouldn't but you can.

Blazer server side lives in this beautiful middleground between the old and the new. The fluent and responsive UI of js frameworks, but the power and control of the old serverside frameworks. It's not perfect, but its close - and its constantly improving.

## What does that mean?

Lets think about front end paradigms that are new and interesting from an SPA point of view:

* Secrets are secret: Private keys can be consumed in the front end - and remain private.
* If its on the server, its in the front end: The filesystem, other machines on the LAN, ports behind the firewall - the lot - its all accessible.
* You control the performance: While you are running every users front end and thus its a heavier load, you get to define the resources - you dont need to worry if an end user has enough cpu to render your massive list in a reasonable time etc - the users browser is just a thin client.
* Its possible to share between sessions/clients: Caching becomes almost irrelevant when you can have data in a static global.

## Do I really want to do that sort of thing in client code?

tl;dr - probably not. That said, if it was okay (and secure) in things like asp.net webforms - its still ok.

Firstly, the front end doesn't need an API anymore - but you probably still do. Often the API is consumed by more than the website. That alone is a reason to not just tear out the API and put all the code in the front end, but you dont really want to manage two versions of the same code.

If youve been a responsible developer and your API Controllers are very thin via a service or even better, by using the mediator pattern (with something like Mediatr) you have a great option; simply call the same code that the API would. This alone can give your app a massive boost. No HTTP requests are required, translation too and from JSON arent required, generally it is a thinner path.

Once this is in place, you then have a decision to make: API controllers/methods which are entirely there to service the front end - can they be removed. The smaller the footprint of an API, the smaller the attack vector.

Once you are in the land of having a service that provides more than the API things get even more interesting. Your functions no longer need to be serializable. Nor do they need to be a "reasonable size".

Lets look at an example. You have an online store. Your fetch products call can return images as images just as a field withing the model if that makes sense. You can fetch the entire product catalog in one fetch - if you need to; something static like this can ... be static. If user a) has viewed the store, the data is already available. The view can still be paginated if that gives a better user experience, but you get the ability to VERY quickly use Linq queries; resort the data, apply filters etc - at lightening speed - theres no API call and theres no DB hit.

If you want to get very tricksy, there are some amazing options open. Lets say this store has a list of recently purchased items. Usually this would either be set on a schedule, once on page load or polled. With everything in the same domain, this could be completely event driven - it doesnt even need to tied to the database.

## Things to consider

Now, if youre like me, this is sounds very whizzy. That said, its not quite as simple as that.
Everything in the previous section can be true - everything being in the same domain, on the same server. But this assumes only one server. If you have a larger site, there is a good chance you are using a scale out pattern - spinning up new servers when load requires - and maybe even spinning down ALL servers at times of zero traffic. That means while sessions can share static variables, not every session can share the SAME session variable. With the last example we gave, the list of purchased items - this means if user A is on server 1 and buys something, then user B who is on server 2 will not receive the event. In some scenarios this is acceptable, but many it is not.

There are work arounds for this. Azure webapps for instance has a local folder - thats actually shared across all instances. This can be watched for changes.

Even if you have to resort to polling the db, this can be done at the server level rather than the session level. This sort of thing goes a long way to offsetting the extra load of hosting - if you have 500 users spread across 5 servers, what would have been 500 API calls before (which likely would have been cached to some extent) you now just have 5 db calls.

A more fundamental concern is browsers without javascript or networks without support for SignalR. In those scenarios, Blazor Serverside just doesnt work.

## Security just got a whole lot easier

Beyond reducing what you expose to the wider world, Blazor Server Side flips the security problem on its head when compared with JS frameworks. As we all know, any code within JS is public. Its run on the client so must be sent to the client. This means you have no ability to hide. Private keys arent private so just arent an option. Obviously this is already a solved problem, but in this world, client code IS private. Private keys ARE private. How does this help?
Well, you need the client to go hit twitter with your dev API key? Thats totally fine. 

## With great power comes great responsibility

with the backend/js front end paradigm you are forced to have a minimul level of seperation between UI and business logic. When dealing with Blazor Server Side, this is no longer enforced. If you so desire, you can have all of you business logic, all db access, all remote API calls - directly in the UI layer. You shouldn't, but you can. Because of this, developers need to be responsible.

## This is a lot to take in. What am I learning here?

Blazor serverside requires you to think about your website from two different angles. These are not new angles, but never before did you really have to consider them at the same time.

Overall, you think about your front end the same way you'd think about a webforms project - or even an API - whilst still considering the front end. Blazors front end is VERY opinionated about how you do things, but with that comes very simple code for doing crazy dynamic things. This coupled with the power that being server side gives you opens up some amazing options if you spend a bit more time thinking about how to architect.

