## How to set up your personal website with GitHub Pages and Jekyll

As a newbie in computer science, I find it necessary to set up my personal website where I can record my knowledge, skills, tips, etc and share my experiences in the learning process. So here it is:)

GitHub is a world-wide open source community for code sharing, they provide a very convenient tool -- [GitHub Pages](https://pages.github.com/) with which we could easily set up our own websites and reference the projects hosted on GitHub.

[Jekyll](https://jekyllrb.com/) is a static site generator perfect for personal site setting up. It is able to transform the plain text into beautiful web pages which allows us to focus more on writing without the hassle of designing and crafting the layout of web pages. 

Here I recorded the main steps it took for me to build my website.
1. create a repository on my GitHub with the same name of my account and GitHub URL suffix, i.e. emilio66.github.io
2. clone the respository to my computer.
3. install Jekyll. [link](https://jekyllrb.com/docs/quickstart/)
> gem install jekyll bundler
4. create the basic site.
> jekyll new emilio66.github.io
5. build the site and preview it locally at http://localhost:4000
> bundle exec jekyll serve

Ok, till now, you have set up the most basic webiste, you can push the code to repo and visit your site on {name}.github.io

6. beautify your webiste.
There are 2 options: 
    1) choose from predefined themes on GitHub [Setting](https://help.github.com/articles/adding-a-jekyll-theme-to-your-github-pages-site-with-the-jekyll-theme-chooser/). 
    2) customize your own style based on open source theme. [Bunch of Jekyll theme you can find here.](https://github.com/topics/jekyll-theme)

I used the most popular one called [minimal mistakes](https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/).
Just several steps:

7. Create if not exists Gemfile, write the following 2 lines:
> source "https://rubygems.org"

gem "github-pages", group: :jekyll_plugins
8. Create if not exists _config.yml, add the following line:
> remote_theme: "mmistakes/minimal-mistakes"
9. Install all the dependencies
> bundle install
10. Preview locally
> bundle exec jekyll serve

I also modified the _config.xml by referencing the default one.

And reference the guide page to write this page. [Link](https://mmistakes.github.io/minimal-mistakes/docs/posts/)
1. Create _post folder if not exists
2. Write the page in markdown format
3. Put the layout in the begining of the post
4. Publish it.

Please correct me if there're any mistakes, thanks for reading!
