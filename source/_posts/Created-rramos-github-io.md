---
title: Created rramos.github.io
date: 2016-09-17 23:57:27
tags:
---
# Started the github pages 

## Intro

I had several blogs in the past that i normaly tend to ignore and stop writting, so this one will be a simple tech/personal blog with entries that i might find usefull in the future to serach quicker.

And as any normal blog i would start by writting the procedure of deploying the blog itself.

The only way i would keep this thing updated is if i have very quick way to edit via terminal a Markdown file and do a command like `post-it` and that's it.

So [hexo](https://hexo.io/) seems like a good solution for me.

Gona try this one out.

So first things first, hosting. I'm cheap bastard so i don't have coold domain yet and i just need some git repo for the blog.

Lets stick with [GitHub Pages](https://pages.github.com/) there git service is pretty owesome so lets try this one out.

**Mental-Note-To-Self:** Get some nice english corrector ;) 

So i've just greated a new repo in github and from the settings i've selected to build a standard GitHub Page.

cloned the dam thing to my computer

`git clone git@github.com:rramos/rramos.github.io.git`

## Installing Hexo

So first things first, let's install the requirements.

It requires:
* Node
* Git

Well, git i allready have since a clonned the repo, node must be installed be it must be the legacy since i'm using Ubuntu Xenial.

```
sudo apt install nodejs-legacy
```

Now starting to install hexo via npm.

```
sudo npm install -g hexo-cli
sudo npm install hexo-deployer-git --save
```

Ok that's it.

## Start writting

So i've run the following command on the repo, this initiates the hexo page structure.

```
hexo init
```

I've added the deploy configuration in _config.yml

```
deploy:
  type: git
  repo: git@github.com:rramos/rramos.github.io.git
  branch: master
  message: [message]
```

I also downloaded a very simple theme, cas'm a simple guy. 

```
cd themes
git clone https://github.com/lotabout/very-simple
```
Thanks @lotabout for the theme by the way.

And installed the theme requirements

```
git clone https://github.com/lotabout/very-simple themes/very-simple
sudo npm install hexo-renderer-sass --save
sudo npm install hexo-renderer-jade --save
```

## Writing

So ... now i suppose i can start creating blog entries, let's start.

```
hexo new post "Created rramos.github.io"
```

I get the output where the md file is and start writting there.

After i have made sure my ssh keys where registered on github i simply deployed with

```
hexo deploy
```

And voilá: https://rramos.github.io  is up and running.

## Final touches 

So hexo only dumps the public part of the blog which makes sense to the git repo. But i want to include all the source and not have separeted repositories for that.

I've created a src dir and copy all the source data there, now i can edit directly that repo, let's take that advantage and remove the default hello world page from the structure guess there migh exist a command for that.

```
$ hexo list post
INFO  Start processing
Date        Title                     Path                                Category  Tags
2016-09-17  Created rramos.github.io  _posts/Created-rramos-github-io.md
2016-09-18  Hello World               _posts/hello-world.md
```

There you are you bastard hello-world, lets get ride of you.

```
hexo clean
rm source/_posts/hello-world.md
hex generate
```

Well guess that's it. There is a lot to explore in hexo from what i can tell, need to check the documentation and understand the community envolvment. Also a quick way to add imagens and other objects and define the quick way to deploy.


