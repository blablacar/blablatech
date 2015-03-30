# BlaBlaTech

BlaBlaCar's Tech Blog — all BlaBlaCar Techies are invited to participate :)

We use [Jekyll](http://jekyllrb.com/docs/home/) for the blog, an easy-to-use way of transform plain text into blogs posts.
Posts are written in [markdown](https://help.github.com/articles/markdown-basics/), it could not be easier!

## Installation

- You need a local webserver in order to run and test the blog
- Clone gh-pages on your machine and configure your localhost accordingly 
- Install Jekyll and the required plugins
  $ gem install jekyll
  $ gem install jekyll-redirect-from
  $ gem install rdiscount

## Tips & Tricks

- Add new authors in the `_config.yml`, it's as easy as this
- Posts are written in [Markdown](http://daringfireball.net/projects/markdown/syntax), can it be any easier?
- Posts (.md files) are to be saved in `_posts`, with the following format: `2011-12-31-new-years-eve-is-awesome.md`
- Posts should be tagged, for instance "Culture", "Tech", "Conference", ...
- Images that belong to a post go into `images`, prefixed with the posts's date (e.g. `2011-12-31-happy.png`)
- Drafts should end up in the folder `_draft`, without a prefixed date (e.g. `new-years-eve-is-awesome.md`)
