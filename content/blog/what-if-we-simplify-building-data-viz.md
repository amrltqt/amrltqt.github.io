+++
title = "What if we simplify Data Viz with React"
date = 2023-04-15
+++

Since I'm working in data analytics industry i'm impressed by the ability of few tools to completely capture the data vizualisation market. PowerBI, Qlik, Tableau and few other old tools are completely eating the cake.

As a software eng, those tools are interesting to explore data but honnestly when it comes to have something reliable in production it's always a pain in the ass. There are no configuration as code available, no real deployment automations and testing good behavior of those application means that you have to do it manually, inspecting each sheets one by one.

Development could be fun, sometimes, but definitely, when it comes to deployement you have to call McGiver.

An idea pop in my mind about that. What if we try to build dashboard with the classical web dev stack? Honnestly, that's not so crazy.

Let's imagine that we build a dashboard with the react / tailwindcss / vitejs stack.

There are few cons:

1. We need to find a way to expose data. There are no way that a web app load data in PG, Snowflake, your TS database or whatever classical database system. Technically, it's seems hard but your sec team will definitely never accept that.

2. A lot of data viz tool propose ways to transform a bit the data. Like grouping by categories, filtering or filling hole in a time series. Those things are not necessarily easy and higly depends on the data exposed.

But to me there are a loot of pro

1. It's a code based application, you build it with current standard and deploy it with classical tools. You immediatly gain correct versionning, clean configuration, CI/CD ..

2. You immediatly win the user experience of a web developper. Put to the trash the clicky, clicky configuration and leverage the precision of any css framework of the market.

3. You can leverage all the existing data viz libraries that exists.

4. You can customize your app without going through a marketplace or a procurement process with your vendor.

For data exposition and manipulation I'm thinking of using apache-arrow to share and unpack data on the web application.

- Generation of the arrow ipc could be delegated to any pandas/python application.
- The arrow ipc dump could be exposed through any blob storage
- I need to think about the way we transform data on front side using apache arrow.

I'll post the progress
