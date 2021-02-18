{:title "Blog The Second"
:layout :post
:tags ["programming" "clojure" "web" "meta" "cryogen" "github" "jekyll"]}

## Reignition
I hadn't touched this blog in a long time, but needing some projects to keep my skills up, I've decided to ressurect it.
However, I didn't want to go back to the standard GitHub Pages Jekyll workflow. I found my relative inability to tinker constraining,
as I didn't know the udnerlying language, namely Ruby. Jekyll is a fine project, but I'm principally a Clojure dev,
and I want to feel like I own my blog, and that means the code that powers it.

## Enter Cryogen
Looking through the static site generators for Clojure, [Cryogen](http://cryogenweb.org/) and [perun](https://perun.io) seemed to be the major players for simple sites. I'd gotten my feet wet with simple projects in both, but never published anything. I like the idea of perun, but its lack of maintainership and needing to learn boot to fully work it, plus incomplete documentation made the decision for me.

When I started in Jekyll, I noted that it should be easy enough to transfer the blog over to a different system. It's just Markdown after all.
I was both right and wrong, and thought writing up a post on how I did the transfer would be good notes for myself in future, and of interest to anyone else seeking to leave Jekyll, or run Cryogen through GitHub Pages.

### 01: Setup
I ran the usual `lein new cryogen unwarysage.github.io` and started tinkering with the config, following Cryogens documentation. Picked out a theme for the moment, tried `:previews` on and off, but by and large I ran with the defaults.

### 02: Migration
Copying over the markdown files into a transitional folder, I went through and converted them by hand, needing only to do eight or so pages. I could have scripted any of these steps, but it wasn't worth it for eight repititions.

First I converted the YAML frontmatter into the Clojure map literal expected by Cryogen. I dropped the date entry, as I don't think I need super precise timestamps, and I'm already dating based on the filename.
Other than changing `categories` to `:tags`, that was all I needed to make the blog compile.

Following that, I needed to make the links work. Cryogen provides a stricter separation of Markdown and template,
 so I didn't have to deal with templating logic, just ordinary markdown links.
  I specifically chose to change `:post-root-uri` to `:posts` to make the links cleaner.

Not caring for my history with Jekyll, I simply set my local git repositories upstream to point at GitHub with `git remote add origin FIXME`, and then force-pushed.
This killed the prior blog, but kept the repository and it's settings.

### 03: Wiring
I read a chunk of the GitHub Pages and Actions documentation, but this next bit took some fumbling, which I will summarize, and not delve into, blow-by-blow.

1. Create `.nojekyll` in the repository root. this tells Github to stop thinking this is a Jekyll project.
2. Ensure that your `:blog-prefix` in Cryogens `config.edn` is set to the empty string. This is so that you could run a Cryogen blog alongside something else and not have the URI clash, but GitHub Pages expect to be flat, with no prefixing.
3. Create `.github/workflows/publish.yml`. If you are using Cryogen, you can just use mine, but you'll need some tweaking otherwise.
4. next, go to your repository on GitHub, and under Settings, enable github pages, set to the root of `gh-pages` branch.
5. Push your changes to `master` and watch the Action run on Github, then go to `https://your-github-username-here.github.io` and bask in the glory.

#### The GitHub Action in detail

```yml
# .github/workflows/publish.yml
name: publish

on:
  push:
    branches:
    - master
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    # checks out a copy of the repository on the runners machine
      - name: Checkout repository
        uses: actions/checkout@master
    # Install Java
      - name: Prepare Java
        uses: actions/setup-java@v1
        with:
          java-version: 1.8
    # Setup leiningen
      - name: Setup Leiningen
        uses: DeLaGuardo/setup-clojure@master
        with:
          lein: 2.9.1
    # Setup NPM
      - name: Setup NPM
        run: sudo apt install nodejs -y & nodejs -v & sudo apt install npm & npm -v
    # Setup Sass
      - name: Setup Sass
        run: sudo npm install sass -g
    # Fetch Lein Deps
      - name: Fetch deps
        run: lein deps
    # Build the blog    
      - name: Build Blog
        run: lein run
    # Deploy to GH-pages
      - name: Deploy Blog
        uses: peaceiris/actions-gh-pages@v3
        with: 
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public/
```
This tells GitHub to run this everytime `master` is pushed to, or by manual request. The action will checkout the repository on the runner, install java, install `lein` (and Clojure), run `apt` to  get `nodejs` and `npm`, then uses `npm` to install the `sass` command. Whatever you do to provide Sass, it needs to be reachable at the path specified in `:sass-path` in `config.edn`.
It will then ask `lein` to pull in it's dependencies, and build the blog. Cryogen puts those in `/public/`. Finally, it uses another preconfigured action to move the files from there to the `gh-pages` branch, creating it if neccesary.

This could definitely be improved by caching leiningens dependencies, and I'll edit this post once I figure that out.

## Conclusion
It's the little details in Software, and getting them exactly right. Deploying to GitHub Pages just makes the feedback loop clunkier, requiring a full commit and push to test. However, I do think the project so far is working quite well. I'm blogging once more, and excited to start tweaking this site to no end, especially now I have reduced the amount of black-box code to just be GitHub Pages deployment system. It's a good feeling.
