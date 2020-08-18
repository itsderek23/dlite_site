---
layout: post
title:  "Understanding DesignOps coming from DevOps"
date:   2020-08-18 05:00:00 -0600
---

As a developer who lived through the advent of DevOps and the squabbling over its meaning, I experienced a bit of déjà vu recently when I stumbled across DesignOps. This post is the Cliff Notes™ version of my research into DesignOps.

Before I begin, here's the Wikipedia definition of [DevOps](https://en.wikipedia.org/wiki/DevOps). I think it is commonly accepted today:

> DevOps is a set of practices that combines software development (Dev) and IT operations (Ops). It aims to shorten the systems development life cycle and provide continuous delivery with high software quality.

This can be shortened to "get shit done" or "release often". DevOps encourages small, incremental releases that are easier to rollback and debug than the massive quarterly release cycles of yore.

## What are common definitions of DesignOps?

[Abstract says](https://www.abstract.com/blog/what-is-designops):

> DesignOps is a dedicated person or team in an organization that focuses solely on enabling the design team to work as well as it possibly can.

Collin Whitehead, Head of Brand at Dropbox [says](https://medium.com/designbetter/getting-started-with-core-models-for-designops-87c5a999acb6):

> “The job of the DesignOps team is to protect the time and headspace of everyone within the design organization – the designers, writers, researchers, and so on – which allows everyone to focus on their respective craft”.

Atlassian [defines](https://www.atlassian.com/blog/inside-atlassian/designops-atlassian-design-studio) DesignOps as:

> "Putting the appropriate tools, instrumentation and processes in place so that we get to ‘learn’ as quickly as possible.”

Adrian Cleave, Director of DesignOps @ AirBnB [says](https://medium.com/airbnb-design/airbnb-designops-2734cf4801b3) this about their DesignOps team:

> Our mission is to provide agility to the whole product organization through centralized tools, systems and services that enhance  speed and quality of execution.

Finally, Almitra Inocenci [says](https://uxdesign.cc/im-sorry-but-design-ops-is-not-new-6c73a1f00e5a):

> It’s a division of people and tasks pertaining to the planning, management, and execution of responsibilities and design process in order to get shit done, whatever the task-at-hand may be, particularly in a design organization.

__My take:__ _DevOps has a definition focused on releasing faster. The definitions of DesignOps tend to be more broad and abstract which makes the term harder to understand. I think part of the reason DevOps is more focused is because dev teams already had a term ([agile development](https://en.wikipedia.org/wiki/Agile_software_development)) that covered the creation portion of releasing software. DevOps depends on this style of development. I don't see the same division for design, so many DesignOps definitions cover everything from the earliest stages (planning) all the way to release._

## What is the origin story of DesignOps?

While it was likely practiced without a name for a while, AirBnB is one of the first brands to discuss the term [starting around 2015](https://medium.com/airbnb-design/airbnb-designops-2734cf4801b3).

## Why DesignOps now?

![dvi](/img/posts/designops/dvi.jpeg)

Via the [Design Value Index Study](https://www.dmi.org/general/custom.asp?page=DesignValue), design-centric public companies returns are more than 200% greater than the S&P 500. That's a damn good reason to prioritize product design. With the increased focus on design, the ratio of developers to designers within an organization has been getting closer. For example, [IBM's developer-to-designer ratio target](https://techcrunch.com/2017/05/31/here-are-some-reasons-behind-techs-design-shortage/) has changed from 72:1 to 8:1. This also doesn't include the increased number of frontend developers that focus on implementing user experiences. It's safe to say the ratio of backend devs to those that touch the user experience is closer than ever before.

More people and faster releases requires more organization and processes, hence the growing importance of DesignOps.

_See Sonja Krogius' post on [Why DesignOps? Why now?](https://blog.nordkapp.fi/why-designops-why-now-c256500595a7) for a detailed look on the growth of DesignOps._

## What are some of the tools AirBnB has developed to support DesignOps?

It's helpful to look at the tooling AirBnB has developed to increase the speed of their design process. They are perhaps the earliest DesignOps evangelist and thus have fairly polished tools. The most significant open source tools AirBnB has shared are:

1. __[Lona](https://github.com/airbnb/Lona)__ - A tool for defining [design systems](https://www.designbetter.co/design-systems-handbook) and using them to generate cross-platform UI code, Sketch files, and other artifacts.
2. __[react-sketchapp](https://github.com/airbnb/react-sketchapp)__ - render React components to Sketch.

__Both of these tools are focused on the chasm that causes the most friction between designs and their release: translating a visual design to code (and back).__

__Lona__'s [background doc](https://github.com/airbnb/Lona/blob/master/docs/overview/background.md) explains how a Sketch-built design system requires manual translation to code for each platform AirBnB supports (web, iOS, Android, and React Native). This is "time consuming and error prone". Lona encodes all of the detail needed to accurately translate from design to code.

__react-sketchapp__ is different than other tools that cross the design-code chasm. Most tools try to go from design to code while react-sketchapp goes in the opposite direction. By working backwards from the source of truth (the design of the deployed app), design systems are able to stay in sync.

## DesignOps should focus on the design-code chasm

> “We’re investing in code as a design tool. Moving closer to working with assets that don’t only include layout and design, but also logic and data. This helps bridge the gap between engineers and designers, thus reducing the need for design specs–or redlines–and the steps between vision and reality”

-[Alex Schleifer](https://airbnb.design/painting-with-code/), head of design at AirBnB

Designers and developers have a chasm to cross: translating design to code (and back). Rather than a change in mindset, it became possible to release high-quality software faster (the goal of DevOps) due to dramatic enhancements in version control (Git), code review tools (GitHub), continuous integration products that automatically run tests, automated code quality products, and error monitoring. If what I saw in DevOps holds true for DesignOps, tools that reduce design/code friction likely outweighs the more abstract, softer side of DesignOps definitions.
