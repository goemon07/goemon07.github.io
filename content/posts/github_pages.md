---
title: "How to create free static website with Hugo and Github Pages"
date: 2021-04-02T08:47:11+01:00
draft: true
# categories: guides
# tags:
#     - github
#     - hugo
#     - website
# keywords:
#     - github
#     - hugo
#     - website
---

So you're struggling to create your **own website**?
Are you looking for some **cheap and clean** solution too? **Free**?

Ok, probably you're a bit too ambitious.

**OR NOT**.

I was in your situation too and now you're visiting this website, which is built on top of the same advices you're going to follow (hopefully!).

If you unfortunately fall into this website, probably you're familiar with the **git** versioning system.
Probably you know about [Github](https://github.com) as well.

Less likely, you know that GitHub can host a website for you.
That's the purpose of [GitHub Pages](https://pages.github.com/)!

Few steps here are described to build up your infrastructure and give in your hands something to work on.

I promise, **you'll enjoy**.

## Step 1: init the repo on Github
In order to embrace the opportunity that GitHub is giving us for free, you neet to init a repository on thier systems. Well, that sounds kinda obvious.
Nevertheless, it's not that evident that the name of your repository cannot be a random one.
In fact, you have to name it as follows: `<your-GitHub-username>.github.io`.

For example, my username is `goemon07`, therefore my linked repository is called `goemon07.github.io`.
Yes, you may have noticed that this is the base URL of your web space as well.

**BINGO!** You're right!

What are you waiting for? Just go and create it!

## Step 2: pick a theme
That's a funny part.
Head to [Hugo Themes page](https://themes.gohugo.io/) and make your choice!
I suggest you to stay simple with the theme.
If it's simple it will be easier to tweak it based on your preferences.
Some cool features you might want to find already in the theme (or at least what I look for):
- **i18n** integration if you want multilanguage website
- **dark/light** theme trigger
- embedded support for **third party links** (i.e.: YouTube videos, Instagram pictures...)

Despite my tips, remembe to pick the best web layout for the porposes of your site.

**Do not focus too much on colors and fonts**, since those can be easily changed!

## Step 3: Install Hugo
Now you actually have to install the bad guy that will power up your `.md` files and translate them into your amazing new website.

Since there's too many ways to install it depending on your OS, your distribution (I hope you're on Linux, right?) and package manager, I just leave you here the [link to the official installation guide of Hugo](https://gohugo.io/getting-started/installing/).

Probably it's enough just to say that is available in all the major package manager for all distros and OSs.

## Step 4: local cloning the repo
It's time to get your working local folder set up.
First step **clone** your plain repo somewhere in your disk.
You can accomplish so by hitting the cloning command like so:

```bash
> git clone git@github.com:<your-user-here>/<your-user-here>.github.io.git
```

Then, it's important to sync the theme repository too, but kind of detatching it from your personal one. That's why submodules are made for!

```bash
> mkdir themes && cd themes
> git submodule add https://github.com/themefisher/vex-hugo.git
```

Once the submodule is up, you probably may want to see your theme running in localhost!
Every hugo theme has a `exampleSite` folder where to run the exact landing page that captured your attenction while selecting the theme.
Since you'll have to work on a copy of those files, I suggest you to copy all the content of `exampleSite` folder into your main one:
```bash
# change dir to your main one, in this case .. to move one step backwards from themes folder
> cd ..
> cp -r themes/<your-theme-folder-here>/exampleSite/* .
> cp -r themes/<your-theme-folder-here>/layouts/* .
```

Once you have all the files, you should be able to show up the live of your website locally:

```bash
> hugo server -D
```

Then head to [localhost:1313](localhost:1313) and see if it's working!

## Step 5: Automatically build and deploy
Wow, almost there!
Now we have to land, somehow, our website online.
*Hugo* just build the website for you out of your `.md` files. What is up to you is to be sure that GitHub can receive those files and expose such in your personal web page.

You can build your website by simply using `hugo` command from your main project dir.
This will create a brand new `public` folder where all your builder store will be stored.

*Cool, ya?*

It  is, but actually we don't need such files in our machine. Instead, such folder should be the one from where GitHub picks from.

*So what?*

What we're going do to let GitHub direcly build and publish the `public` folder into a separate branch of your repo as soon as you `git push` your last commits.

*Sounds like a plan!*

### Step  5.1: Exclude `public` folder from main branch
As already explained, we don't really need our `public` folder in our `master` branch since it's just the product of the build that the *Action* will do for us.
Therefore, it is worth to add `public` into the `.gitignore` file in the project directory.

If you already have a `.gitignore` file, just attach the folder in a new line. If not, create the file and write down the folder too.

I.e., this is how my `.gitignore` looks like:

```bash
Thumbs.db
*/uncompressed/
*/gimp/
.dist
.tmp
.sass-cache
.log
node_modules
package-lock.json
public/
```

### Step 5.2: Setup new Deploy key
A new deploy key will enable our machine, where the ssh key is stored, to being authorized to fire the Action that we're going to enplace.
In order to generate the ssh key, place your terminal in the `~/.ssh` folder and then fire:
```bash
> ssh-keygen -t rsa -b 4096 -C "$(git config user.email)" -f gh-pages -N ""
```
This will generate a pair of new public/private `gh-pages` key files.

Now we need share our public key with GitHub so that it will recognize us.

Head to your repository settings and click on **Deploy Key** section to add a new one. Then copy it into your clipboard using:

```bash
> xclip -selection clipboard < gh-pages.pub
```

And `ctrl+v` here:
![GitHub Deploy Key screenshot](/images/posts/github_pages/deploy.png)

Remember to enable the *Allow write access* toggle be able to `git push`.

Do not leave *GitHub settings* page, since we need to set up  a new *secret*!
Get into the *Secrets* section and click on *New repository secret* button.

![GitHub Secret screenshot](/images/posts/github_pages/secret.png)

Remember to name the new secret as `ACTIONS_DEPLOY_KEY` or remember how you name it for the next step.
This time, in the big text box you have to share the previously created private key. Copy it into your clipboard using the following command from the `~/.ssh` folder:

```bash
> xclip -selection clipboard < gh-pages
```
... and paste it into the form.

### Step 5.3: Setup GitHub Action

It is possible to automate the process of deploying changes as soon as the commit has been pushed into the GitHub repository.
In fact, leveraging *Actions*  we can:
- build our Hugo project
- publish the `/public` folder into a separate branch
- let *GitHub Pages* watch that specific branch to

```yaml
name: github pages

on:
  push:
    branches:
      - master  # Set a branch to deploy

jobs:
  deploy:
    runs-on: ubuntu-18.04
    steps:
      - uses: actions/checkout@v2
        with:
          submodules: true  # Fetch Hugo themes (true OR recursive)
          fetch-depth: 0    # Fetch all history for .GitInfo and .Lastmod

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: 'latest'

      - name: Build
        run: hugo -D

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          deploy_key: ${{ secrets.ACTIONS_DEPLOY_KEY }}
          publish_branch: gh-pages
          publish_dir: ./public
```

### Step 5.4: Setup GitHub Pages
*Almost there!!!*

Last missing piece before the `git push`, I promise.
Now our `public` directory should be delivered into the `gh-pages` branch automatically.
We just miss to let GitHub Pages know that the source is in that specific branch and not in the `master` one.

We can simply tweak this from the repo settings, main section (named "*Options*").
Scroll down until you reach "*GitHub Pages*".
In the dropdown menu of "*Source*" section, select `gh-pages` branch and **Save** it!

![GitHub Pages setup screenshot](/images/posts/github_pages/pages.png)

## Step 6: Finally, `git push`!
Well, know you're done. Seriously.
Type `git push` from your project folder, check your action processing from the *Actions* section in your GitHub repo page and enjoy your brand new website!

![git push](/images/posts/github_pages/git.gif)

## Conclusions
I hope you find this guide useful.
GitHub pages is really a game changer when you want to publish your presentation website or simply have a place where to sponsor your small business.

Thanks for being with me till the end!
