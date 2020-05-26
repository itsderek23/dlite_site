---
layout: post
title:  "Machine learning deserves its own flavor of Continuous Delivery"
date:   2020-04-17 08:00:00 -0600
author: Derek Haynes
description: An ML project ties together large datasets, a slow training process without clear pass/fail acceptance tests, and contains multiple types of deliverables. By comparison, a classical software project just contains code and has a single deliverable (an application). Thankfully, it's possible to create a Machine Learning-specific flavor of Continuous Delivery (CD4ML) for non-enterprise organizations with existing tools today.
hero_image: /img/posts/cd4ml/cd4ml_tools.png
excerpt: Things feel slightly different - but eerily similar - when traveling through Canada as an American. The red vertical lines on McDonald's straws are a bit thicker, for example. It's a lot like my travels through the world of data science as a software engineer.
canonical_url: https://booklet.ai/blog/continuous-delivery-machine-learning-cd4ml/
---

![i love lucy](/img/posts/cd4ml/i_love_lucy.jpg)
<p class="small text-muted">
Continuous Delivery for Machine Learning, <a href="https://www.youtube.com/watch?v=HnbNcQlzV-4">in practice</a>.
</p>


Things feel slightly different - but eerily similar - when traveling through Canada as an American. The red vertical lines on McDonald's straws are a bit thicker, for example. It's a lot like my travels through the world of data science as a software engineer.

__While the data science world is magical, I'm homesick for the refined state of [Continuous Delivery (CD)](https://en.wikipedia.org/wiki/Continuous_delivery) in classical software projects when I'm there.__ What's Continuous Delivery?

Martin Fowler [says](https://www.martinfowler.com/bliki/ContinuousDelivery.html#footnote-when) you're doing Continuous Delivery when:


> 1. Your software is deployable throughout its lifecycle
2. Your team prioritizes keeping the software deployable over working on new features
3. Anybody can get fast, automated feedback on the production readiness of their systems any time somebody makes a change to them
4. You can perform push-button deployments of any version of the software to any environment on demand

For a smaller-scale software project deployed to [Heroku](http://heroku.com), Continuous Delivery (and its hyper-active sibling [Continuous _Deployment_](https://www.martinfowler.com/bliki/ContinuousDelivery.html#footnote-when)) is a picture of effortless grace. You can setup up [CI](https://devcenter.heroku.com/articles/heroku-ci) in minutes, which means your unit tests run on every push to your git repo. You get [Review Apps](https://devcenter.heroku.com/articles/github-integration-review-apps) to preview the app from a GitHub pull request. Finally, every `git push` to master automatically deploys your app. All of this takes less than an hour to configure.

Now, why doesn't this same process work for our tangled balls of yarn wrapped around a _single_ `predict` function? Well, the world of data science is not easy to understand until you are deep in its inner provinces. __Let's see why we need [CD4ML](https://martinfowler.com/articles/cd4ml.html) and how you can go about implementing this in your machine learning project today.__

## Why classical CD doesn't work for machine learning

I see 6 key differences in a machine learning project that make it difficult to apply classical software CD systems:

### 1. Machine Learning projects have multiple deliverables

Unlike a classical software project that solely exists to deliver an application, a machine learning project has multiple deliverables:

* ML Models - trained and serialized models with evaluation metrics for each.
* A web service - an application that serves model inference results via an HTTP API.
* Reports - generated analysis results (often from notebooks) that summarize key findings. These can be static or interactive using a tool like [Streamlit](https://www.streamlit.io).

Having multiple deliverables - each with different build processes and final presentations - is more complicated than compiling a single application. Because each of these is tightly coupled to each other - and to the data - it doesn't make sense to create a separate project for each.

### 2. Notebooks are hard to review

Code reviews on git branches are so common, they are [baked right into GitHub](https://github.com/features/code-review/) and other hosted version control systems. GitHub's code review system is great for `*.py` files, but it's terrible for viewing changes to JSON-formatted files. Unfortunately, that's how notebook files (`*.ipynb`) are stored. This is especially painful because notebooks - not abstracting logic to `*.py` files - is the starting point for almost all data science projects.

### 3. Testing isn't simply pass/fail

<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/0MDrZpO_7Q4" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
<p class="small text-muted">Elle O'Brien covers the ambigous nature of using CI for model evaluation.</p>

It's easy to indicate if a classical software git branch is OK to merge by running a unit test suite against the code branch. These tests contain simple true/false assertions (ex: "can a user signup?"). Model evaluation is harder and often requires a data scientist to manually review as evaluation metrics often change in different directions. This phase can feel more like a code review in a classical software project.

### 4. Experiments versus git branches

Software engineers in classical projects create small, focused git branches when working on a bug or enhancement. While branches are lightweight, they are different than model training experiments as they usually end up being deployed. Model experiments? Like fish eggs, the vast majority of them will never see open waters. Additionally, an experiment likely has very few changes code changes (ex: adjusting a hyper-parameter) compared to a classical software git branch.

### 5. Extremely large files

Most web applications do not store large files in their git repositories. Most data science projects do. Data almost always requires this. Models may require this as well (there's also little point in keeping binary files in git).

### 6. Reproducible outputs

It's easy for any developer on a classical software project to reproduce the state of the app from any point in time. The app typically changes in just two dimensions - the code and the structure of the database. It's hard to get to a reproducible state in ML projects because many dimensions can change (the amount of data, the data schema, feature extraction, and model selection).

## How might a simple CD system look for Machine Learning?

![cd4ml tools](/img/posts/cd4ml/cd4ml_tools.png)
<p class="small text-muted">
Some of the tools available to make CD4ML come to life.
</p>

In the excellent [Continuous Delivery for Machine Learning](https://martinfowler.com/articles/cd4ml.html), the [Thoughtworks](https://www.thoughtworks.com/) team provides a [sample ML application](https://github.com/ThoughtWorksInc/cd4ml-workshop) that illustrates a CD4ML implementation. Getting this to run involves a number of moving parts: GoCD, Kubernetes, DVC, Google Storage, MLFlow, Docker, and  the EFK stack (ElasticSearch, FluentD, and Kibana). Assembling these ten pieces is approachable for the enterprise clients of Throughworks, but it's a mouthful for smaller organizations. __How can leaner organizations implement CD4ML?__

Below are my suggestions, ordered by what I'd do first. The earlier items are more approachable. The later items build on the foundation, adding in more automation at the right time.

### 1. Start with Cookiecutter Data Science

> Nobody sits around before creating a new Rails project to figure out where they want to put their views; they just run rails new to get a standard project skeleton like everybody else.

<p class="small text-muted">
via <a href="https://drivendata.github.io/cookiecutter-data-science/">Cookiecutter Data Science</a>
</p>

When I view the source code for an existing Ruby on Rails app, I can easily navigate around as they all follow the same structure. Unfortunately, most data science projects don't use a shared project skeleton. However, it doesn't have to be that way: start your data science project with [Cookiecutter Data Science](https://github.com/drivendata/cookiecutter-data-science) rather than dreaming up your own unique structure.

Cookiecutter Data Science sets up a project structure that works for most projects, creating directories like `data`, `models`, `notebooks`, and `src` (for shared source code).

### 2. nbdime or ReviewNB for better notebook code reviews

![nbdiff](/img/posts/cd4ml/nbdiff-web.png)
<p class="small text-muted">nbdime is a Python package that makes it easier to view changes in notebook files.</p>

It's hard to view changes to `*.ipynb` in standard diff tools. It's trivial to conduct a code review of a `*.py` script, but almost impossible with a JSON-formatted notebook. [nbdime](https://nbdime.readthedocs.io/en/latest/) and [ReviewNB](https://www.reviewnb.com/) make this process easier.

### 3. dvc for data files

<pre>
dvc add data/
git commit -ma "Storing data dir in DVC"
git push
dvc push
</pre>
<p class="small text-muted">After installing dvc and adding a remote, the above is all that's required to version-control your <code>data/</code> directory with dvc.
</p>

It's easy to commit large files (esc. data files) to a git repo. Git doesn't handle large files well: it slows things down, can cause the repo to grow in size quickly, and you can't really view diffs of these files anyway. A solution gaining a lot of steam for version control of large files is [Data Version Control (dvc)](https://github.com/iterative/dvc). dvc is easy to install (just use `pip`) and its command structure mimics git.

__But wait - you're querying data direct from your data warehouse and don't have large data files?__ This is bad as it virtually guarantees your ML project is not reproducible: the amount of data and schema is almost guaranteed to change over time. Instead, dump those query results to CSV files. Storage is cheap, and dvc makes it easy to handle these large files.

### 4. dvc for reproducible training pipelines

dvc isn't just for large files. As Christopher Samiullah [says](https://christophergs.com/machine%20learning/2019/05/13/first-impressions-of-dvc/#pipelines) in _First Impressions of Data Science Version Control (DVC)_:

> Pipelines are where DVC really starts to distinguish itself from other version control tools that can handle large data files. DVC pipelines are effectively version controlled steps in a typical machine learning workflow (e.g. data loading, cleaning, feature engineering, training etc.), with expected dependencies and outputs.

You can use [`dvc run`](https://dvc.org/doc/tutorials/get-started/pipeline) to create a Pipeline stage that encompasses your training pipeline:

<pre>
dvc run -f train.dvc \
          -d src/train.py -d data/ \
          -o model.pkl \
          python src/train.py data/ model.pkl
</pre>
<p class="small text-muted">
<code>dvc run</code> names this stage <code>train</code>, depends on <code>train.py</code> and the <code>data/</code> directory, and outputs a <code>model.pkl</code> model.
</p>

Later, you run this pipeline with just:

<pre>
dvc repro train.dvc
</pre>

Here's where "reproducible" comes in:

* If the dependencies are unchanged - `dvc repo` simply grabs `model.pkl` from remote storage. There is no need to run the entire training pipeline again.
* If the dependencies change - `dvc repo` will warn us and re-run this step.

If you want to go back to a prior state (say you have a `v1.0` git tag):

<pre>
git checkout v1.0
dvc checkout
</pre>
<p class="small text-muted">Running <code>dvc checkout</code> will restore the <code>data/</code> directory and <code>model.pkl</code> to is previous state.
</p>

### 5. Move slow training pipelines to a CI cluster

In [Reimagining DevOps](https://www.youtube.com/watch?v=0MDrZpO_7Q4&list=PLVeJCYrrCemgbA1cWYn3qzdgba20xJS8V&index=6), Elle O'Brien makes an elegant case for using CI to run model training. I think this is brilliant - CI is already used to run a classical software test suite in the cloud. Why not use the larger resources available to you in the cloud to do the same for your model training pipelines? In short, push up a branch with your desired experiment changes and let CI do the training and report results.

Christopher Samiullah [also covers moving training to CI](Samiullah), even providing a link to a [GitHub PR](https://github.com/ChristopherGS/dvc_test/pull/1) that uses `dvc repo train.dvc` in his CI flow.

### 6. Deliverable-specific review apps

![netlify deploy preview](/img/posts/cd4ml/github-statuses-deploy-previews.png)
<p class="small text-muted">
Netlify, a PaaS for static sites, lets you view the current version of your site right from a GitHub pull request.
</p>

Platforms like Heroku and Netlify allow you to view a running version of your application right from a GitHub pull request. This is fantastic for validating things are working as desired. These are great for presenting work to non-technical team members.

Do the same using GitHub actions that start Docker containers to serve your deliverables. I'd spin up the web service and any dynamic versions of reports (like something built with [Streamlit](http://www.streamlit.io)).

## Conclusion

Like eating healthy, brushing your teeth, and getting fresh air, there's no downside to implementing Continuous Delivery in a software project. We can't copy and paste the classical software CD process on top of machine learning projects because an ML project ties together large datasets, a slow training process that doesn't generate clear pass/fail acceptance tests, and contains multiple types of deliverables. By comparison, a classical software project contains just code and has a single deliverable (an application).

Thankfully, it's possible to create a Machine Learning-specific flavor of Continuous Delivery ([CD4ML](https://martinfowler.com/articles/cd4ml.html)) for non-enterprise organizations with existing tools (git, Cookiecutter Data Science, nbdime, dvc, and a CI server) today.

__CD4ML in action__? _I'd love to put an example machine learning project that implements a lean CD4ML stack on GitHub, but I perform better in front of an audience. If I get enough subscribers below, I'll put this together. Please subscribe!_

{% include mailchimp.html %}

## Elsewhere

* [Continuous Delivery for Machine Learning](https://martinfowler.com/articles/cd4ml.html) - this blog post (and the CD4ML acronym) wouldn't exist without the original work of Daniel Sato, Arif Wider, and Christoph Windheuser.
* [First Impressions of Data Science Version Control (DVC)](https://christophergs.com/machine%20learning/2019/05/13/first-impressions-of-dvc/#pipelines) - this is a great technical walkthrough migrating an existing ML project to using DVC for tracking data, creating pipelines, versioning, and CI.
* [Reimagining DevOps for ML by Elle O'Brien](https://www.youtube.com/watch?v=0MDrZpO_7Q4&list=PLVeJCYrrCemgbA1cWYn3qzdgba20xJS8V&index=6) - a concise 8 minute YouTube talk that outlines a novel approach for using CI to train and record metrics for model training experiments.
* [How to use Jupyter Notebooks in 2020](https://ljvmiranda921.github.io/notebook/2020/03/06/jupyter-notebooks-in-2020/) - a comprehensive three-part series on architecting data science projects, best practices, and how the tools ecosystem fits together.
