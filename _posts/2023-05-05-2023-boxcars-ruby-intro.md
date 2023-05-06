---
layout: post
title:  "Using Boxcars - the lightweight Ruby Langchain alternative - to query a Rails DB with natural language"
date: 2023-05-06 05:00:00 -0600
---

You may have heard of [Langchain](https://github.com/hwchase17/langchain), the Python library for creating LLM-powered apps with nearly 35k GitHub stars. Despite the large following, [Langchain can be difficult to use](https://news.ycombinator.com/item?id=35820931) when you want to go deeper than "hello world" tutorials. This experience (and my background as a Rubyist) led me to [Boxcars](https://github.com/BoxcarsAI/boxcars), a Langchain-inspired Ruby gem but with fewer abstactions. 

You might be asking: why venture outside the Python ML ecosystem? Well, building apps with LLMs doesn't require Ruby equivalents for NumPy, SciPy and Pandas. Rather than working with numbers, I find that most of my time is spent manipulating string templates and interacting with outside systems (like realtime search). The readability of Ruby is great for this use case.

In this post, I'll use Boxcar to query my Rails database using natural language. I'll take a look at how Boxcars creates ChatGPT prompts, handles errors, and how it compares to Langchain's `SQLDatabaseChain` to solve the same problem.


## Querying ActiveRecord with natural language

The Ruby ecosystem is Rails-centric, so it's great to see that Boxcars plays well with Rails apps out of the box. Just add the `boxcars` gem to your `Gemfile`, set the `OPENAI_ACCESS_TOKEN` to your OpenAI API key, and you can start querying your database with natural language inside `rails console`:

```ruby
boxcar = Boxcars::ActiveRecord.new
boxcar.run "How many users?"
{"status":"ok","answer":33,"explanation":"Answer: 33","code":"User.count"}
 => 33 
```

You may have used ChatGPT to generate code for you, but this goes a step beyond: it executes the code! 

To see how the magic happens, I'll call `Boxcars.configuration.log_prompts = true` and re-run the above code to inspect the generated ActiveRecord prompt:

<pre>
>>>>>> Role: system <<<<<<                                                   
You are a Ruby on Rails Active Record code generator                         
>>>>>> Role: system <<<<<<                                                   
Given an input question, first create a syntactically correct Rails Active Record code to run, then look at the results of the code and return the answer. Unless the user specifies in her question a specific number of examples she wishes to obtain, limit your code to at most 5 results.
Never query for all the columns from a specific model, only ask for the relevant attributes given the question.
Also, pay attention to which attribute is in which model.                    
                                                                             
Use the following format:
Question: ${{Question here}}
ARChanges: ${{Active Record code to compute the number of records going to change}} - Only add this line if the ARCode on the next line will make data changes.
ARCode: ${{Active Record code to run}} - make sure you use valid code
Answer: ${{Final answer here}}

Only use the following Active Record models: []
Pay attention to use only the attribute names that you can see in the model description.
Do not make up variable or attribute names, and do not share variables between the code in ARChanges and ARCode
Be careful to not query for attributes that do not exist, and to use the format specified above.
Finally, try not to use print or puts in your code
>>>>>> Role: user <<<<<<
Question: How many users?
</pre>

The model responds with:

<pre>
ARCode: User.count
</pre>

If you copy and paste the prompt above into ChatGPT you should see a very similar response. 

## How does the ActiveRecord Boxcar execute the query? 

The ActiveRecord Boxcar checks to see if `ARCode` is in the response. If it is (and after some security checks) it executes the code returning the result of the ActiveRecord query.

## What about adjusting queries if the first attempt is malformed?

Let's say I ask Boxcar to run a query for an ActiveRecord model that does not exist. What does it? Does it immediately exit, attempt to fix the issue, or just raise an exception?

```ruby
boxcar.run "how many ClassDoesNotExist records were created this year?"
```

ChatGPT responds with a valid-looking query:

```ruby
ARCode: `ClassDoesNotExist.where("created_at >= ?", Time.zone.now.beginning_of_year).count`
```

The Boxcar runs the query and captures the exception:

```ruby
Error while running code: uninitialized constant Boxcars::ActiveRecord::ClassDoesNotExi ...
```

It will then re-try up to 3 additional times. Notice how Boxcar appends (1) the code that was excecuted (2) the error that resulted from running the query:

```
...
>>>>>> Role: user <<<<<<
Question: how many ClassDoesNotExist records  were created this year?
>>>>>> Role: assistant <<<<<<
ARCode: `ClassDoesNotExist.where("created_at >= ?", Time.zone.now.beginning_of_year).count`
>>>>>> Role: user <<<<<<
ARCode Error: uninitialized constant Boxcars::ActiveRecord::ClassDoesNotExist - please fix "ARCode:" to not have this error
```

ChatGPT then returns a response, but it lacks the `ARCode` section:

```
I apologize for that. It seems like there is no `ClassDoesNotExist` model in the list of available models. Please let me know which model you would like to use instead.
```

The boxcar appends this error to the prompt and re-runs:

```
>>>>>> Role: assistant <<<<<<
I apologize for that. It seems like there is no `ClassDoesNotExist` model in the list of available models. Please let me know which model you would like to use instead.
>>>>>> Role: user <<<<<<
Your answer wasn't formatted properly - try again. I expected your answer to start with "ARChanges:" or "ARCode:"
```

ChatGPT attempts to help, but we're not going to get anywhere:

```
I apologize for the mistake. Here is the correct format:

ARCode: `ModelName.where("created_at >= ?", Time.zone.now.beginning_of_year).count`

Please replace `ModelName` with the name of the model you would like to use.
```

This is smart usage of updating the ChatGPT prompt with additional context around errors. 

## How does it do with complex queries?

At first, it struggled in my development environment and I almost wrote it off as another ML "hello world" demo that quickly fails when you try to take it farther. Then I realized that the default list of models and their attributes in the prompt is likely to be empty (or contain just a small number of columns) [per this SO question](https://stackoverflow.com/questions/516579/is-there-a-way-to-get-a-collection-of-all-the-models-in-your-rails-app). After running `Rails.application.eager_load!` and re-initializing my boxcar I was very impressed. It correctly executed queries like these:

1. Which org has the most users?
2. Order the orgs by the number of users in each org and show the orgs with the most users. list the org id, name, and number of users.
3. How many users were created by month?

## How does the ActiveRecord Boxcar compare to Langchain's SQL Toolkit?

This is a very small sample size, but the ActiveRecord Boxcar provided more accurate results for me than the Langchain's `SQLDatabaseChain`. For example, it returned a result when I provided an invalid table name and returned an incorrect value in query 2 above due to a missing join.

## TL;DR

If you are a Rubyist, don't let Langchain's large following sway you away from trying Boxcars when creating an LLM application. If you're like me, you'll enjoy the smaller footprint, fewer abstractions, and a faster timeline to production usage (assuming you already have a deployed Rails app) that Boxcars offers.