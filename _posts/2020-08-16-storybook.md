---
layout: post
title:  "Storybook - a beautiful library for your web components"
date:   2020-08-16 05:00:00 -0600
published: false
---

Your web app is composed of individual components (often with multiple states) that can be difficult to test. For example, a navigation header may have multiple states:

* Displaying the avatar of a user if logged in and a "sign in" link if not
* Adding a banner if a free trial is coming to a close shortly
* Adding a notice if parts of the service are not working correctly
* Displaying additional features for admin users

It's very difficult to verify each of these states render correctly. I'll start up a local version of a web app, load a page in my browser, and override values (ie set `admin = true`) to view states. However, this is fragile and slow. Enter [Storybook](https://storybook.js.org/), an open source UI component explorer that lets you develop UI components in isolation.

Here's a look at my initial experience using Storybook.

## Who is Storybook for?

Storybook is designed for frontend developers that are already using a Javascript framework to create components and CSS to style them. However, there are several secondary users:

* Designers - verify the design of UI components and their states are true to your designs.
* Project Managers - quickly QA UI component changes without loading the entire web app locally.
* Backend developers & ops - alleviate the need for frontend devs, designers, and project managers to keep a local dev stack maintained if they primary just need access to UI components.

## Setting up Storybook

Storybook is installed inside an existing application. In my test of Storybook, I used `create-react-app` to setup a simple React app (see a detailed [React + Storybook tutorial](https://www.learnstorybook.com/intro-to-storybook/react/en/get-started/) for more info) then installed Storybook via `npx -p @storybook/cli sb init`. In addition to React, Storybook supports Vue, Angular, Ember, and more frameworks. However, the official docs and tutorials appear to be more extensive for React than other frameworks.

Once installed, you can start a local Storybook server via `yarn storybook`. This command opens a browser tab to `http://localhost:6006` and is filled with a number of example React components and their associated stories (more on stories later). Checking out these examples is a good place to start. The Storybook app is well-designed with a clean esthetic.

Storybook has an addon system to extend its functionality. Starting with version 6.0.0, [essential addons are pre-installed](https://medium.com/storybookjs/zero-config-storybook-66e7c4798e5d).

Storybook doesn't magically import your existing React components. However, it's not a huge amount of work to have a component appear in Storybook. This is also a process you can perform incrementally, adding your most important components first.

## Creating stories

At first, I was expecting to see my custom component appear without any stories. However, I realized this didn't make much sense as most components require some sort of default parameter values to appear. For example, my alert component needs some text to render.

A story describes an interesting state of component. For my alert box, a `Basic` and `Error` state make sense to me as stories.

## Importing a React component into Storybook

My example React app contains an alert component in `src/components/Alert.js` directory. This displays important notifications to the user of different types (basic, error, etc). To import this component, I just follow the following steps:

* Create a `src/components/Alert.stories.js` file with at least one story. Stories follow the [Component Story Format](https://storybook.js.org/docs/react/api/csf).
* Restart the storybook server. Restarting is only required when adding a component, not updating an existing component.

## Adding stories

My button has multiple states: primary, secondary, and large. These different states are called stories in Storybook. I'll create a story....

## Importing data

## TL;DR
