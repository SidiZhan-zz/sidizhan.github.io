---
title: how do i set up this blog site
date: 2018-10-24 21:27:41
tags:
---
how to set up a hexo blog site to be visited via *.github.io?
======
```bash

```

I tried to set up hexo once months ago, but didn't touch it until recently. I screwed it so now I have to set it up again. In order to remember all the steps, I decide to record the process down here.

install
-----
First of all you should install node, npm and yarn. And then hexo itself (a git repository as starter).
```bash
npm install -g hexo-cli
```

Go to a directory/workspace that you want to put your site, for instance `vonwunderland`. Use npm to install all the required packages in `package.json`.
```bash
hexo init vonwunderland
cd vonwunderland
npm install
```

Now if you run the server you will be able to visit your site with a hello world post locally (localhost:4000).
```bash
hexo server
```

new post
-----
Create a new post by:
```bash
hexo new "how do i set up this blog site"
```
then edit it with markdown format, in any text editor (me, visual studio code with markdown preview plug-in). The server will auto track the changes.

to github
-----
Now you want to deploy this onto github so that everyone can visit your blog site. So go to github and create a repo named `yourname.github.io` (my is `sidizhan.github.io`). Ignore the node. Then locally, install the deployer.
```bash
npm install hexo-deployer-git --save
```

Copy the url for repo and paste it in `_config.yml`, under `deploy:`, final edition is like:
```bash
deploy:
  type: git
  repo: git@github.com:SidiZhan/sidizhan.github.io.git
  branch: master
  message: "{{ now('YYYY-MM-DD HH:mm:ss') }}"
```

Then, you can deploy:
```bash
hexo deploy
```
and visit the website at [https://sidizhan.github.io/](https://sidizhan.github.io/)!

Tips: 
-----
it may take minutes to update the setting in github.io, so if you can only visit home page by specifying the index.html (https://sidizhan.github.io/index.html), or the new posts won't show up, just be patient and wait.

in a nut shell
-----
Don't forget the process to publish a new post:
```bash
cd vonwunderland
hexo new "new post"
hexo generate
hexo deploy
```
you need to `generate` before  `deploy` if you want to push the new post into github. 


references
-----
[hexo](https://hexo.io/docs/configuration)

[Create-Host-Blog-for-free-with-Hexo-Github](https://malekbenz.com/blog/2016/09/10/Create-Host-Blog-for-free-with-Hexo-Github)
