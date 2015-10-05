# Setting up your development environment

This document will guide you through the steps necessary to setting up a development environment for **JMI Labs**!

This guide assumes that you will be setting up your environment on a new OS X installation (10.10+). 

## Easy Buttton 

The **thoughtbot** team has a script that will automagically turn your vanilla OS X install into an awesome web development machine. 

In your terminal type, 

~~~bash
bash <(curl -s https://raw.githubusercontent.com/thoughtbot/laptop/master/mac) 
~~~

### What it sets up

* **Bundler** for managing Ruby libraries
* **Exuberant Ctags** for indexing files for vim tab completion
* **Foreman** for managing web processes
* **hub** for interacting with the GitHub API
* **Heroku Toolbelt** for interacting with the Heroku API
* **Homebrew** for managing operating system libraries
* **ImageMagick** for cropping and resizing images
* **Node.js** and **NPM**, for running apps and installing JavaScript packages
* **Postgres** for storing relational data
* **Qt** for headless JavaScript testing via Capybara Webkit
* **Rbenv** for managing versions of Ruby
* **RCM** for managing company and personal dotfiles
* **Redis** for storing key-value data
* **Ruby Build** for installing Rubies
* **Ruby** stable for writing general-purpose code
* **The Silver Searcher** for finding things in files
* **Tmux** for saving project state and switching between projects
* **Zsh** as your shell

## Outline

The goal is to setup and install the following components: 
 
* [Homebrew](##Homebrew)
* [git](##Git)
* [Ruby](##RVM)
* [Rails](##Rails)
* [PostgreSQL](##PostgreSQL)

We can then install optional componenets 

* [zsh](##zsh)
* [oh my zsh](##oh-my-zsh)
* [tmux](##tmux)
* Sublime Text 2

***

Throughout this guide we will be interacting with the Terminal application on OS X. To access terminal press `⌘ + Space` to open Spotlight search and type `terminal`.

## Homebrew

Homebrew installs the stuff you need that Apple didn’t.

1. In your terminal type: 
`ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"`

2. During the installation, your computer will inform you that Xcode command line tools is required. Follow the prompt to install the necessary dependencies. *If Xcode command line tools does not install by default type* `xcode-select --install`  

3. To verify the installation of homebrew was successful type: `brew doctor`. You should see something the resulting output: `Your system is ready to brew.`

## Git

Git is a free and open source distributed version control system designed to handle everything from small to very large projects with speed and efficiency.

1. In your terminal type `brew install git`. 

2. To verify the installation, in your terminal type `git --version`.

3. Approximate expected result: `git version 2.6.0`.

Now we will configure git,

1. In your terminal type `git config --global user.name "Your Actual Name"`. 

2. And then type, `git config --global user.email "Your Actual Email"`.

3. To verify your configuration type: `git config -l`.

## RVM and Ruby

RVM is short for **Ruby Version Manager**. RVM makes it easy to install different versions of Ruby on your computer and manage them as well

1. To install RVM, type this in your terminal `\curl -L https://get.rvm.io | bash -s stable`.

2. Close your terminal and then re-open it. Now, lets see if RVM was loaded properly: `type rvm | head -n 1`

3. Now install Ruby with: 
    ```bash
    rvm use ruby --install --default   
    ruby -v
    ```

## Rails

Ruby on Rails® is an open-source web framework that’s optimized for programmer happiness and sustainable productivity. It lets you write beautiful code by favoring convention over configuration.

1. In your terminal type, `gem install rails --no-ri --no-rdoc`.

2. To verify your installation, type `rails --version`.

   Approximate expected result: `Rails 4.2.4`.

## PostgreSQL

PostgreSQL is a powerful, open source object-relational database system. It has more than 15 years of active development and a proven architecture that has earned it a strong reputation for reliability, data integrity, and correctness.

1. In your terminal type, `brew install postgresql`.

2. To verify your installation, type `brew list | grep "postgresql"`. You should see `postgresql` as an output. 

## zsh

Zsh is a shell designed for interactive use, although it is also a powerful scripting language.

1. In your terminal type, `brew install zsh`. 

2. To verify your installation, type `brew list | grep "zsh"`. You should see `zsh` as an output. 

## oh-my-zsh

Oh My Zsh is an open source, community-driven framework for managing your zsh configuration. 

1. In your terminal type,
~~~
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
~~~

**Themes**

Zsh has  over one hundred themes now bundled. Most of them have [screenshots](https://wiki.github.com/robbyrussell/oh-my-zsh/themes) on the wiki. Check them out!

## tmux

1. In your terminal type, `brew install tmux`.

2. To verify your installation, type `brew list | grep "zsh"`. You should see `tmux` as an output. 

## Sublime Text 2 


1. Download Sublime Text 2 [here](http://www.sublimetext.com/2). 




