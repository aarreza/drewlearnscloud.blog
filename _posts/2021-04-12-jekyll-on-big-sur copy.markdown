---
layout: post
title:  "Installing Jekyll on OSX Big Sur"
date:   2021-04-12 20:21:21 -0400
categories: [Software]
tags: web
permalink: /web
---
I aim to have a concise _'how-to install Jekyll'_ on the latest macOS 11.x as I found a few minor nuances around using the Ruby version that comes pre-installed on a Mac. 

In my case I will be installing Jekyll on a fresh install of macOS Big Sur and won't be using any Ruby version managers such as [rbenv](https://github.com/rbenv/rbenv). If you're in software development, I would highly suggest looking at Ruby Package and Version Management [here](https://www.ruby-lang.org/en/documentation/installation/#package-management-systems).  The default shell I'll be using is `zsh`, which Apple has decided to switch to from `bash` since [macOS Catalina](https://scriptingosx.com/2019/06/moving-to-zsh/).

[macOS 11.x Big Sur](https://www.apple.com/ca/newsroom/2020/11/macos-big-sur-is-here/) has been out since Nov. 12, 2020 and I've decided to always keep tabs with the latest. After installing the prerequisites and running `$ jekyll -v`, I receive an error message: 

{% highlight console %}
-bash: jekyll: command not found
{% endhighlight %}

## **Prerequisites**

Here are the prerequisites to install Jekyll on macOS 11.x Big Sur _(non M1 chip)_. This also works with macOS version:
- macOS Catalina 10.15
- macOS Mojave 10.14

_If you've got an [M1](https://www.apple.com/ca/mac/m1/) chip, be aware of open issue in Jekyll's Github [#8576](https://github.com/jekyll/jekyll/issues/8576)._

To [check your processor](https://support.apple.com/en-us/HT211814), click the Apple icon on the menu bar and open _About your Mac_.

You may follow the links below to get some context on the tools we'll be using.

1. [Ensure you update macOS](https://support.apple.com/en-ca/guide/mac-help/mchlpx1065/mac)
2. [Homebrew](https://brew.sh/)
2. [Xcode Command Line Tools](https://developer.apple.com/xcode/)
3. [Ruby](https://www.ruby-lang.org/en/documentation/faq/1/) v2.4.0 or higher for Jekyll
4. [Bundler](https://bundler.io/)
5. [Git](https://git-scm.com/)

## **Ensure you update macOS**
Before you begin, make sure that you have the latest Apple update for your macOS version. Go to _System Preferences_, then _Software Update_.

## **Homebrew**
If you haven't installed Homebrew yet, read on. If not you may skip this section. Confirm that you have Homebrew installed by running `ls /usr/local` in `Terminal` app.

{% highlight console %}
ls /usr/local
Homebrew
{% endhighlight %}

Confirm you are ready to use Homebrew by running `brew doctor`. Your output should be `Your system is ready to brew`.

To install Homebrew, run:
{% highlight console %}
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
{% endhighlight %}

Allow a few minutes to complete this installation. 

## **Xcode Command Line Tools**
Run the following to install Xcode Command Line Tools:
{% highlight console %}
xcode-select --install
{% endhighlight %}

## **Ruby v2.4.0 or higher**
Jekyll is written in Ruby and it is helpful to understand the following terminology.

**Gems** - code you can include in Ruby projects, you may add multiple gems packages to your project

**Gemfile** - list of gems used by your site. Every Jekyll site has a Gemfile in the main root folder. 

**Bundler** - a gem that installs all gems in your Gemfile. 

### **Installing Ruby**
If you'd like to manage multiple Ruby versions, there's a number of version managers you can check out `rbenv`, `rvm`, `chruby`. I won't be developing anything else at least for now so I'll stick with updating the preinstalled Ruby (v2.6.3) on my Mac. To check your Ruby version, type this command `ruby -v`. 

We're ready to brew. Let's update Ruby by entering:

{% highlight console %}
brew install ruby
{% endhighlight %}

Once the installation is complete you will see this output. 
{% highlight console %}
If you need to have ruby first in your PATH run:
  echo 'export PATH="/usr/local/opt/ruby/bin:$PATH"' >> ~/.zshrc

For compilers to find ruby you may need to set:
  export LDFLAGS="-L/usr/local/opt/ruby/lib"
  export CPPFLAGS="-I/usr/local/opt/ruby/include"
{% endhighlight %}

The first line is important to note, as whenever you run a command like `ruby` or `gem`, your operating system will search through a list of directories which reside in the environment variable called `PATH`. 

Execute that first line to add the Ruby installation to your `PATH` environment variable.

{% highlight console %}
echo 'export PATH="/usr/local/opt/ruby/bin:$PATH"' >> ~/.zshrc
{% endhighlight %}

The `~/.zshrc` is a script that is run whenever you start `zsh` Terminal.

To verify that the code executed correctly. 

{% highlight console %}
cat ~./zshrc
{% endhighlight %}

You should find the following line appended.

{% highlight console %}
export PATH="/usr/local/opt/ruby/bin:$PATH"
{% endhighlight %}

At this point, quit and relaunch your terminal, and verify everything is working with `ruby -v`. It should be updated to `ruby 3.0.1p64`.

You will now be able to manage Ruby Gems. Verify gem version using the command `gem -v`. The output will be `3.2.16`.

Finally, update your Ruby gems. 

{% highlight console %}
gem update --system
{% endhighlight %}

## **Bundler**
Install Bundler:

{% highlight console %}
gem install bundler
{% endhighlight %}

## **Git**
If you haven't used Git before, you will need to install it as to be able to upload your Jekyll site to Github. Read about [Git](https://git-scm.com/). 

Install Git:
{% highlight console %}
brew update
brew install git
{% endhighlight %}

It is always a good practice to run `brew update` so that you get the latest version of software prior to installing something.

Verify your Git installation by running `git --version`. My version is `git version 2.24.3 (Apple Git-128)`.

## **Jekyll**
To launch your first site, choose a location and create a new folder.

{% highlight console %}
cd ~
mkdir jekyll-first-website
cd jekyll-first-website
{% endhighlight %}

### **Create a Gemfile**
Inside `jekyll-first-website`, enter `bundle init` to create a `Gemfile`. 

### **Finally Install Jekyll**
Enter the following to add the necessary Gems for Jekyll. 
{% highlight console %}
bundle add jekyll
{% endhighlight %}

One thing to note is Jekyll won;'t run on Ruby 3.0 [#8523](https://github.com/jekyll/jekyll/issues/8523)

In order to fix this, run `bundle add webrick` to install this library.

After adding Jekyll, you can verify your `Gemfile` updated with the line `gem "jekyll", "~> 4.2.0"`

You are now ready to create the scaffolding for the site. We use the `--force` parameter since our folder isn't empty then run the `bundle install` separately because Jekyll gets confused if the Gemfile already exists. 

{% highlight console %}
bundle exec jekyll new --force --skip-bundle .
bundle install
{% endhighlight %}

To verify the installed gems enter `ls /usr/local/lib/ruby/gems/3.0.0/gems`

You're ready to serve your site,while under your working directory `jekyll-first-website` enter `bundle exec jekyll serve` and visit your [localhost](https://en.wikipedia.org/wiki/Localhost) at http://127.0.0.1:4000/

Prior to committing your code to [Source Control](https://git-scm.com/book/en/v2/Getting-Started-About-Version-Control), you'll want to edit your `.gitignore` file. 

{% highlight console %}
# Ignore metadata generated by Jekyll
_site/
.sass-cache/
.jekyll-cache/
.jekyll-metadata

# Ignore folders generated by Bundler
.bundle/
vendor/
{% endhighlight %}