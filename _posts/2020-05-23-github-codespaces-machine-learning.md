---
layout: post
title:  GitHub Codespaces for Machine Learning
date:   2020-05-26 06:53:05 -0600
hero_image: /img/posts/codespaces/github_codespaces_screenshot.png
description: Learn how to give potential contributors to your ML project a smooth onramp by providing a pre-configured IDE environment via GitHub Codespaces.
---

![github_codespaces_screenshot](/img/posts/codespaces/github_codespaces_screenshot.png)

<p class="text-muted small" style="margin-top:-10px">
Give potential contributors to your ML project a smooth onramp by providing a pre-configured IDE environment.
</p>

Attempting to contribute to an open-source machine learning project requires a leap-of-faith. Is there a list of dependencies? Where is the training data? Is there a pre-trained model? Should I just bail and watch Lego Masters instead? __It can take hours to hotwire an ML project.__

This is why I was very excited to hear about the BETA release of [GitHub Codespaces](https://github.com/features/codespaces) at [Satellite 2020](https://githubsatellite.com/). __With Codespaces, contributors can spin up a ready-to-go GitHub project-specific dev environment in the cloud.__ In this post, I'll show how to give potential contributors a graceful start by configuring Codespaces for an ML project.

## Doesn't Google Colab already do this?

Online notebook services like [Google Colab](https://colab.research.google.com/notebooks/welcome.ipynb) provide a _Jupyter notebook environment_ in the cloud. These are great for proof-of-concepts, but a [notebook-only environment falls down when you move beyond exploration](https://towardsdatascience.com/the-case-against-the-jupyter-notebook-d4da17e97243) for several reasons:

* __Sharing code across notebooks__ - evolved ML projects use purpose-specific notebooks that have shared components (like loading data). It's not easy keeping your notebooks [DRY](https://en.wikipedia.org/wiki/Don't_repeat_yourself) in a notebook-only environment. This results in a bloated, inconsistent code base.
* __Explicit vs. implied dependencies__ - notebook environments come pre-installed with a number of Python packages. While this is nice when proving a concept, relying on these implicit dependencies is fragile and breaks a key principle of [12 Factor apps](https://12factor.net/dependencies).
* __Versioning data__ - An ML model's output is dependent on both code and training data. Both need to be in version control to ensure the results are reproducible. This isn't possible with a notebook-only environment.
* __Beyond the notebook__ - While somewhat obvious, if you want to provide other avenues to use your model (like an HTTP web service or CLI) you can't make those available in a notebook-only environment.

__With GitHub Codespaces, you have both a Jupyter notebook environment and a full IDE experience (code editor & terminal access).__ You don't have to resort to ugly workarounds for the above issues.

## The ML foundation - whisk

```bash
$ pip install whisk
$ whisk create demo
$ cd demo
$ source venv/bin/activate
```

<p class="text-muted small" style="margin-top:-10px">
<a href="http://docs.whisk-ml.org">whisk</a> creates a data science-flavored version of a Python project structure.
</p>

It's easy to run an ML project within Codespaces when it has a solid structure. __The [project structure](https://docs.whisk-ml.org/en/latest/project_structure.html) in this tutorial was generated using [whisk](http://docs.whisk-ml.org), an open-source ML project framework that makes collaboration, reproducibility, and deployment “just work”.__ I've been developing whisk with Adam Barnhard of [Booklet.ai](https://booklet.ai).

[whisk](http://docs.whisk-ml.org) generates a Pythonic project structure that makes it easy to share code across notebooks, version control data with [DVC](https://dvc.org), distribute a model as a Python package, and spin up an HTTP web service for the model. whisk extends many of the ideas put forth in [Cookiecutter Data Science](https://drivendata.github.io/cookiecutter-data-science/). __Because a whisk ML project behaves just like a Python project, it's easy to spin up a Codespaces environment.__

## The ML project - Real or Not? NLP with Disaster Tweets

<img src="/img/posts/codespaces/twitter_logo.png" style="float:left;max-width: 100px;margin-right: 20px" />

I'll be referencing [Real or Not? NLP with Disaster Tweets](https://github.com/whisk-ml/disaster_tweets), an ML project that trains a Tensorflow-backed Keras model to detect if a tweet is about a legitimate disaster. Besides the raw trained model, the project contains:

* [A notebook](https://github.com/whisk-ml/disaster_tweets/blob/master/notebooks/explore.ipynb) to explore the dataset
* [A Flask HTTP web app](https://github.com/whisk-ml/disaster_tweets/blob/master/app/main.py) to invoke the model
* [A CLI command](https://github.com/whisk-ml/disaster_tweets/blob/master/src/disaster_tweets/cli/main.py) to invoke the model
* A versioned dataset and pipeline via [DVC](https://dvc.org)
* A CLI command to distribute the model as a Python package

Outside of the notebook, these pieces require a _full development environment_, not _just a notebook environment_.

## Just add a devcontainer.json file

With Codespaces, you customize the environment with a [`devcontainer.json`](https://github.com/microsoft/vscode-dev-containers/blob/master/containers/python-3/.devcontainer/devcontainer.json) file. You can place a `devcontainer.json` file in the top-level of your project or inside a `.devcontainer` directory.

[This project's devcontainer.json file](https://github.com/whisk-ml/disaster_tweets/blob/master/.devcontainer/devcontainer.json) looks like this:

{% highlight js %}
{
  "image": "mcr.microsoft.com/vscode/devcontainers/python:3.7",
  "extensions": ["ms-python.python"],
  "postCreateCommand": "bash .devcontainer/post_create.sh",
  "forwardPorts": [5000],
  "context": "..",
  "settings": {
    "terminal.integrated.shell.linux": "/bin/bash",
    "python.pythonPath": "./venv/bin/python"
  },
}
{% endhighlight %}

Here's why these settings are used:

* __image__ - Use the [Python3.7 Docker image](https://github.com/microsoft/vscode-dev-containers/tree/master/containers/python-3#python-3). The project only relies on Python being installed - all of the remaining dependencies are listed in the `requirements.txt` file inside the project.
* __extensions__ - While the [ms-python.python](https://marketplace.visualstudio.com/items?itemName=ms-python.python) extension includes several nice-to-haves, the primary purpose of the extension in this project is that it installs Jupyter Notebooks. When opening a `*.ipynb` file, Codespaces uses a notebook editor view letting contributors run and modify notebook cells.
* __postCreateCommand__ - After the container is created,  [the `post_create.sh` script](https://github.com/whisk-ml/disaster_tweets/blob/master/.devcontainer/post_create.sh) initializes the whisk environment and grabs data files via DVC.
* __forwardPorts__ - Forwards port 5000 (the port used by the Flask app) so the HTTP web service can be used.
* __context__ - This is the path that the Docker build should be run from relative to the `devcontainer.json` file. As I've included `devcontainer.json` inside a folder, I need to access the parent directory which contains the top-level project.
* __settings__ - The key/values here are [VSCode settings](https://code.visualstudio.com/docs/getstarted/settings#_settings-file-locations). These set the terminal to use bash and provide the path to the Python interpreter. [These are the available Python extension settings](https://code.visualstudio.com/docs/python/settings-reference).

View the [devcontainer.json reference](https://code.visualstudio.com/docs/remote/containers#_devcontainerjson-reference) on the Visual Studio website for a list of all available configuration settings.

{% include mailchimp.html %}

## Running the project inside Codespaces

![download](/img/posts/codespaces/open_codespaces_button.png)

We're ready to start. From the [GitHub project](https://github.com/whisk-ml/disaster_tweets), click the "Code" button and create a new codespace. After a couple of minutes the project environment will be created.

### The notebook

![notebook](/img/posts/codespaces/open_notebook.png)

In the sidebar explorer, double-click on "notebooks > explore.ipynb". This loads the iPython notebook and initializes the Python extension. Click the fast-forward button to execute all of the cells.

### The terminal

Open a terminal session from the sidebar:

<video controls id="new_terminal" loop style="width:100%">
 <source src="/img/posts/codespaces/new_terminal.mp4" type="video/mp4">
Your browser does not support the video tag.
</video>
<script>
 document.getElementById("new_terminal").playbackRate = 1.5;
</script>

#### Invoke the model

[whisk](http://docs.whisk-ml.org) sets up a CLI to invoke the model. You can run this from the command line:

<video controls id="disaster_tweets_predict_terminal" style="width:100%">
 <source src="/img/posts/codespaces/disaster_tweets_predict_terminal.mp4" type="video/mp4">
Your browser does not support the video tag.
</video>
<script>
 document.getElementById("disaster_tweets_predict_terminal").playbackRate = 3;
</script>

In the above example, the model believes that this is most likely a real disaster.

#### Use the web service

[whisk](http://docs.whisk-ml.org) also sets up a Flask web service to invoke the model. You can use the web service from Codespaces (mind blown). To start the web app, run `whisk app start` inside the terminal. To view the web app in your browser, click the remote explorer icon and "Port 5000" under forwarded ports:


<video controls id="flask_app" loop style="width:100%">
 <source src="/img/posts/codespaces/disaster_tweets_flask.mp4" type="video/mp4">
Your browser does not support the video tag.
</video>
<p class="small text-muted">
You can even view web apps inside GitHub Codespaces.
</p>

<script>
 document.getElementById("flask_app").playbackRate = 3.0;
</script>

#### See the pipeline for training the model

You might be curious: how is model is trained? What files does the training stage generate? As this project uses [DVC](https://dvc.org) to version control the training pipeline it's easy to inspect the training flow from the terminal.

From the terminal you can view the stages in the pipeline with `dvc pipeline show`:

```
dvc pipeline show --ascii train.dvc

+----------------------+             +--------------------+
| download_dataset.dvc |             | download_glove.dvc |
+----------------------+             +--------------------+
                    ***               ***                   
                       ***         ***                      
                          **     **                         
                        +-----------+                       
                        | train.dvc |                       
                        +-----------+
```

DVC shows that there are three stages and that the _train_ stage depends on two download stages.

You can view the outputs from all stages with the `-o` flag:

```
dvc pipeline show -o train.dvc
data/raw/train.csv
data/raw/test.csv
data/raw/sample_submission.csv
data/raw/glove.6B.100d.txt
src/disaster_tweets/artifacts/model.h5
src/disaster_tweets/artifacts/tokenizer.pickle
```

Training generates the `model.h5` and `tokenizer.pickle` files.

#### No need to run training

One of the great things about using DVC to version control the training pipeline is you only need to re-run the training stage if any of the stage dependencies aren't up-to-date (ex: you modify the training script or raw training data). Training takes about 20 minutes in this project. With the `dvc repro` command we know that training does not need to be re-run as nothing has changed. This delivers a much faster bootstrapping experience:

<video controls id="dvc_repro_terminal" style="width:100%">
 <source src="/img/posts/codespaces/dvc_repro_terminal.mp4" type="video/mp4">
Your browser does not support the video tag.
</video>
<script>
 document.getElementById("dvc_repro_terminal").playbackRate = 2;
</script>

## Future enhancements

There are some things I'd like to improve about this setup in the future:

* __git commits fail via the UI__ - Currently I can't commit from the UI as it doesn't appear the git commands are run within the project's venv. My commits fail as I call `dvc` in a pre-commit hook and dvc is installed in the venv. This hasn't been a blocker for me as I use the terminal anyway for git.  
* __Clearer ready state__ - Codespaces appears to let you use the IDE before everything is loaded. For example, I can open a notebook prior to to the Python extension being installed. When this happens, I'll see a bunch of scary warnings (ex: Python isn't installed). Waiting a bit clears the warnings and then I can execute the notebook.

## Summary

With a [whisk](https://whisk-ml.org)-structure ML project and a [`devcontainer.json` file](https://help.github.com/en/github/developing-online-with-codespaces/configuring-codespaces-for-your-project), you've created an easy onramp for potential contributors. [Codespaces](https://github.com/features/codespaces) is a good option when your project has evolved beyond the proof-of-concept, single notebook stage of an ML project.

## Reference

* [GitHub Codespaces Docs](https://help.github.com/en/github/developing-online-with-codespaces)
* [Disaster Tweets - Real or Not? GitHub repo](https://github.com/whisk-ml/disaster_tweets)
* [Whisk ML Project Framework](https://docs.whisk-ml.org/en/latest/)
* [Data Version Control (DVC)](https://dvc.org)
