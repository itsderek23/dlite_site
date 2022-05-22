# Derek Haynes Public Site

This is the static site for [dlite.cc](https://dlite.cc), powered by [Jekyll](https://jekyllrb.com/), a Ruby static site generator. It is used for the public-facing parts of Booklet (homepage, blog, etc).

## Blog instructions

To create a new blog article:

* `git checkout -b blog/[INSERT TITLE]` - create a Git branch
* Add a new file to to the `_posts` directory w/the same `DATE-TITLE.md` format as other posts.
* Copy and update the [front matter](https://jekyllrb.com/docs/front-matter/) from the top of an existing post.
* Write the post in Markdown format.
* Images - place images in `/img/posts/[INSERT TITLE]`. Reference these like `![INSERT DESCRIPTION](/img/posts/[INSERT TITLE]/image.png)`.
* When done, `git push` the branch and create a PR on GitHub. Netlify will build a preview. When ready to publish, merge the PR and the update will be deployed to https://booklet.ai.

See the [Jekyll docs on posts](https://jekyllrb.com/docs/posts/) for more details.

## Dependencies

* Ruby 2.6.2 (Netlify uses this version. Other Rubies are slower to build.)

## Development setup instructions

1. Install Ruby
2. Install [RVM](http://rvm.io/)
3. `git clone` this repo
4. `cd` into the downloaded git repo
5. `rvm install 2.6.2` - installs the required Ruby version via RVM. This will take a bit.
6. `bundle` to install Ruby dependencies.

## Starting the app

`foreman start` - runs Jekyll on http://localhost:4000.

## Deploying updates

Pushing the `master` branch automatically deploys the updates to `Netlify`.
