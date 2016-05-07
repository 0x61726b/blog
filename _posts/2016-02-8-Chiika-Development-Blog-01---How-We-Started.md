---
layout: post
title: "Chiika Development Blog 01 - How We Started"
modified: 2015-11-23 00:01:26 +0200
tags: [chiika,programming]
image:
  feature:
  credit: 
  creditlink: 
comments: 
share: 
---
Hey everyone,we have decided that we would be writing our development experience of Chiika since blogging could play a great role in creating an open source software.In this post I'm going to go over the thought process of creating *Chiika*.

##Chiika ?

We have tried really hard to come up with a name,i mean really. But we literally get frustruted when trying to come up with something unique,because I think we are really lazy about that stuff.Here is a list of names we have selected as canditates

![Trello List](http://i.imgur.com/jPQfql4.png)

So after debating for hours,we have decided that we should solve this with democracy.

![strawpoll](http://i.imgur.com/S8as2fk.png)

Whoops,it didn't work too...

In the end we finally agreed on **Chiika**

If 散(chi) is combined with 花(ka) it becomes "flower",thus our logo.

Oh,I didn't mention what Chiika *really* is. Chiika is a cross platform MAL tracker/scrobbler. We decided we'd do something like this for the community and ourselves.
We thought that the application we were using at the time were handy enough already,but we wanted more. Since we are programmers and have the passion to create new stuff,it wasn't hard to decide if we were doing it or not.

After hours of talking what we'd do and what not, we made a list of features,planning ahead and getting started in the development.


We had really trouble choosing what tools/technologies to work with. Since I'm a C++ guy, it was for sure that I'm creating the API in C++.For UI related stuff,without awareness of current technologies, we picked Qt to start with.

We went a lot of way with Qt, until the day comes when I got sick of it.The data binding in Qt was just too painful to deal with.And on top of it,we were using Qml which is an awful choice for desktop applications. If you start a default blank application in QML, it will eat 25 Mbs of memory regardless of the platform. What the hell?
Upon learning about so called *Web tech* , I knew that we *have to* go with it.Since we have HTML/Css background a little, it would be hell of a better choice than QML than we were completely unfamiliar.

Here are some drafts we have done with Qt/Qml

![](http://i.imgur.com/hG9xswH.png)

![](http://i.imgur.com/Dx7lULZ.png)

![](http://i.imgur.com/ISaHgYF.png)

![](http://i.imgur.com/Pegrsdt.png)

![](http://i.imgur.com/8HPxsae.png)
