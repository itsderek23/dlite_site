---
layout: post
title:  "Meet whisk: a lightweight data science project framework with magical results"
date:   2020-06-02 05:00:05 -0600
description: TODO
---

Like an eager prospector rushing to pan for gold - supplies and dignity be damned - I used to dive into a data science project with just a Jupyter Notebook and a bit of drool collecting in my Civil War era beard. However, that boundless, explore-at-all-cost drive soon backfired. In short time, I'd have an unorganized notebook, duplicate code, plenty of ugly hacks, and a project that's difficult to share, reproduce, and deploy as an ML model.

I'm not alone. __Rather than [using Python best practices](https://docs.python-guide.org/writing/structure/) for structuring a project, most data science projects evolve from a notebook.__ Starting with a notebook is logical: who knows if this will go anywhere? What I need is something that lets me focus on data science but adds _just enough_ structure to make the project easy to share.

That's why I'm happy to introduce __[whisk](http://docs.whisk-ml.org), an open-source data science project framework that makes collaboration, reproducibility, and deployment “just work”.__ whisk is actually kind of boring, but that's the point. It combines a data science-flavored Python project structure - the same project structure that's been almost unchanged over a decade - with a suite of lightweight tools. It lets you setup your data science projects like a software engineer without studying to be one.

Now, why should you care about structuring your data science project like a software project? You're not a software engineer after all. Well, when this structure is applied, it lets you do magical things in a pure, battle-hardened Python way. __In the spirt of "show, don't tell" I'm going to show how to create a whisk data science project. Then, I'll show how to bootstrap, use, distribute, and deploy an existing DS project that uses whisk.__

## Creating a project

Much like `django-admin startproject` creates the structure for a Django web app, whisk sets up a [project directory structure](https://docs.whisk-ml.org/en/latest/project_structure.html) and environment for you. You start by installing whisk. Open a terminal and type `pip install whisk`. Next, use the [`whisk create`](https://docs.whisk-ml.org/en/latest/cli_reference.html#whisk-create) command to create a new data science project:

<script id="asciicast-uXW9s6FtkmbxNQQIdtDRLFYII" data-rows=30 src="https://asciinema.org/a/uXW9s6FtkmbxNQQIdtDRLFYII.js" async></script>

The [initial structure](https://docs.whisk-ml.org/en/latest/project_structure.html) contains an end-to-end machine learning example that saves a model to disk and shows how to later load the model and generate a prediction. whisk doesn't force you to use a particular ML framework or train a model in a certain way. The structure it creates is uniformly applicable across data science projects (well, at least the ones I've worked on).

You can follow the [quick tour](https://docs.whisk-ml.org/en/latest/tour_of_whisk.html) in the whisk docs to get orientated on the initial scaffolding. Rather than walking through a trivial example, lets take a look at project built with whisk.

{% include mailchimp.html %}

## Exploring an existing project

[Real or Not: NLP with Disaster Tweets](https://github.com/whisk-ml/disaster_tweets) is an ML project that trains a Tensorflow-backed Keras model to predict whether a tweet is about a real disaster or not. It was structured with whisk. It's painful bootstrapping most data science projects, but that's not the case with this whisk. Just run `git clone` and `whisk setup` to get started:

<script id="asciicast-BvEd0bjpKabHRzSraIFOvgGeb" data-rows=20 src="https://asciinema.org/a/BvEd0bjpKabHRzSraIFOvgGeb.js" async></script>

whisk includes [DVC](https://dvc.org/), an open-source version control system for machine learning projects. This project leverages DVC to store the large training data set, version control the training pipeline, and store the generated model artifacts. __DVC is core to a great bootstrapping experience: there's no need to re-train the model when you initially checkout the project (which can take > 20 minutes) as DVC verifies nothing has changed in the training pipeline.__

### Invoke the model via the CLI

whisk projects include [click](https://click.palletsprojects.com/en/7.x/), a Python package for creating beautiful command line interfaces. This is pre-configured to invoke the model. The default command name is the whisk project name:

<script id="asciicast-pfzn5UQOoBCJ7GgoVUJnByToa" data-rows=10 src="https://asciinema.org/a/pfzn5UQOoBCJ7GgoVUJnByToa.js" async></script>

### Run the notebook

Notebooks are placed within the `notebooks/` directory. As the Python package is installed locally, you can easily call functions from the project's `src/` directory from the notebook. This reduces the amount of noise in a notebook and makes it easy to use shared code across multiple notebooks.

### Distribute the model as a Python package

__You can distribute the model as a Python package, letting anyone with Python installed on their computer use the model.__ A whisk project contains default `setup.py` and `MANIFEST.in` files to get you started. To build the package, type `whisk package dist`:

<script id="asciicast-T93exVYzcEfixDseoBNQb5mRN" data-rows=10 src="https://asciinema.org/a/T93exVYzcEfixDseoBNQb5mRN.js" async></script>

This is easy as a whisk project is structured like the Python packages you already use.

### Run the web service

A Flask app is included in the `app/` directory of every whisk project. It's ready to serve predictions for the project's model. To start the web service, type `whisk app start`:

<script id="asciicast-wX72KSJg06qwDRal2dpIvIYmR" data-rows=10 src="https://asciinema.org/a/wX72KSJg06qwDRal2dpIvIYmR.js" async></script>

Use `curl` to send a request to the Flask app and generate a prediction:

<script id="asciicast-J3QOAugHYbyOqZRpjsQp9CUH3" data-rows=13 src="https://asciinema.org/a/J3QOAugHYbyOqZRpjsQp9CUH3.js" async></script>

The flask app autoreloads when the source code in your project changes. This makes the developing and debugging cycle very fast.

### Deploy the model to Heroku

The vast majority of ML models do not require a Kubernetes cluster. __Kubernetes is expensive and hard to maintain - why not just deploy to [Heroku](https://heroku.com) like thousands of Python devs have before you?__ A whisk project is pre-configured to deploy the web service to Heroku. You can deploy the model with a single command, [`whisk app create`](https://docs.whisk-ml.org/en/latest/cli_reference.html#whisk-app-create):

<script id="asciicast-Z0H72T7ADU9e4grJs9jlLqtJ3" data-rows=20 src="https://asciinema.org/a/Z0H72T7ADU9e4grJs9jlLqtJ3.js" async></script>

Heroku is likely free for your initial proof-of-concept and certainly less expensive and less work than maintaining a Kubernetes cluster.

## What about Cookiecutter Data Science?

Does whisk smell a lot like [Cookiecutter Data Science](https://drivendata.github.io/cookiecutter-data-science/)? You have a good nose! My first attempts at structuring a DS project used it. whisk takes the well-regarded structure created by Cookiecutter DS and sprinkles on some magic:

* __Graceful upgrades__ - It's difficult, manual work upgrading a previously generated Cookiecutter DS project to the latest project structure. In fact, the most-commented open issue on the [Cookiecutter](https://github.com/cookiecutter/cookiecutter) package is [_How to update a project with changes from its cookiecutter?_
](https://github.com/cookiecutter/cookiecutter/issues/784) and has been open since 2016. Unlike Cookiecutter projects, whisk projects include the [whisk package](https://pypi.org/project/whisk/) as a dependency. This shifts plumbing code (like [CLI commands](https://docs.whisk-ml.org/en/latest/cli_reference.html#) and [helper functions](https://docs.whisk-ml.org/en/latest/key_concepts.html#helper-functions)) out of the project, reducing the amount of code and providing a clearer upgrade path as new features are added to whisk. whisk follows the same dependency model used popular web framework libraries like Django and Flask.
* __Environment configuration__ - Besides the directory structure, whisk also sets up a Python3 venv, initializes a Git repo for the project, creates an iPython kernel, [and more](https://docs.whisk-ml.org/en/latest/autoapi/whisk/setup/index.html#whisk.setup.setup). These are all standard activities I found myself doing over-and-over inside a CookieCutter DS-generated project.
* __Packaging__ - The code within the `src/` directory of a whisk project can easily be packaged and distributed. This doesn't work with Cookiecutter DS.
* __DVC__ - Cookiecutter DS includes some ad-hoc functions for handling data. whisk shifts these to [DVC](https://dvc.org), a dedicated library.
* __Deployment__ - whisk adds a Flask app for model serving and a CLI command to deploy the web service to Heroku.

With 3.5k GitHub stars, Cookiecutter DS is a popular, well-received open-source project. If you are deciding between Cookiecutter DS and whisk, we're already winning the war against sloppy data science.


## Beliefs

Now that you've seen how whisk works, I'd like to share the beliefs that drove the creation of whisk. whisk is not for everyone. However, if the beliefs below resonate with you, it might make sense for your next DS project:

* **A Reproducible, collaborative project is a solved problem for classical software** - We don't need to re-invent the wheel for machine learning projects. Instead, we need guide rails to help data scientists structure projects without forcing them to also become software engineers.
* **A notebook is great for exploring, but not for production** - A data science notebook is where experimentation starts, but you can't create a reproducible, collaborative ML project with just a `*.ipynb` file.
* **Optimize for debugging** - 90% of writing software is fixing bugs. It should be fast and easy to debug your model logic locally. You should be able to search your error and find results, not sift through custom package source code or stop and restart Docker containers.
* **Python already has a good package manager** - We don't need overly abstracted solutions to package a trained ML model. A properly structured ML project lets you distribute the model via `pip`, making it easy for _anyone_ to benefit from your work.
* **Version control is a requirement** - You can't have a reproducible project if the code and training data isn't in version control.
* **Docker is an unsteady foundation** - when we [explicitly declare and isolate dependencies](https://12factor.net/dependencies), we don't need to rely on the implicit existence of packages installed in a Docker container. Python has solid native tools for dependency management.
* **Kubernetes is overkill** - very few web applications require the complexity of a container-orchestration system. Your deployed model is no different. Most models can run on boring, reliable technology.

## Get started!

Follow the [whisk quick tour](https://docs.whisk-ml.org/en/latest/tour_of_whisk.html) to get started and make collaboration, reproducibility, and deployment “just work” on your next DS project.

## whisk resources

* [Documentation](https://docs.whisk-ml.org)
* [whisk on GitHub](https://github.com/whisk-ml/whisk)
* Example projects
  * [Disaster Tweets](https://github.com/whisk-ml/disaster_tweets) - A Tensorflow-backed Keras model that predicts which tweets are about real disasters and which ones are not.
  * [Bike Image Classifier](https://github.com/whisk-ml/bike_image_classifier_tensorflow) - A Tensflow-backed classifier to determine if an image is of a Mountain bike or a Road bike.
