# Setting up your development environment

This document will guide you through the steps necessary to setting up a development environment for **JMI Labs**!

This guide assumes that you will be setting up your environment on a new OS X installation (10.10+). 

## Easy Buttton 

The **thoughtbot** team has a script that will automagically turn your vanilla OS X install into an awesome web development machine. 

In your terminal type, 

~~~bash
bash <(curl https://raw.githubusercontent.com/bwittenbrook3/dev-setup/master/.laptop.local > .laptop.local)
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
 
* [Homebrew](#homebrew)
* [git](#git)
* [Ruby](#ruby)
* [Rails](#rvm-and-ruby)
* [PostgreSQL](#postgresql)
* [Node.js & NPM](#nodejs--npm)

We can then install optional componenets 

* [zsh](#zsh)
* [oh my zsh](#oh-my-zsh)
* [tmux](#tmux)
* [Sublime Text 2](#sublime-text-2)

***

Throughout this guide we will be interacting with the Terminal application on OS X. To access terminal press <kbd>⌘</kbd>+<kbd>space</kbd> to open Spotlight search and type `terminal`.

## Homebrew 

Homebrew installs the stuff you need that Apple didn’t.

1. In your terminal type: 
~~~
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
~~~

2. During the installation, your computer will inform you that Xcode command line tools is required. Follow the prompt to install the necessary dependencies. 

*If Xcode command line tools does not install by default type* `xcode-select --install`  

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

3. Now install Ruby with: `rvm use ruby --install --default`.

4. To verify your installation, type `ruby -v`.  

   Approximate expected results: `ruby 2.2.1`.

## Rails

Ruby on Rails® is an open-source web framework that’s optimized for programmer happiness and sustainable productivity. It lets you write beautiful code by favoring convention over configuration.

1. In your terminal type, `gem install rails --no-ri --no-rdoc`.

2. To verify your installation, type `rails --version`.

   Approximate expected result: `Rails 4.2.4`.

## PostgreSQL

PostgreSQL is a powerful, open source object-relational database system. It has more than 15 years of active development and a proven architecture that has earned it a strong reputation for reliability, data integrity, and correctness.

1. In your terminal type, `brew install postgresql`.

2. To verify your installation, type `brew list | grep "postgresql"`. You should see `postgresql` as an output. 

## Node.js & NPM

Node.js® is a JavaScript runtime built on Chrome's V8 JavaScript engine. Node.js uses an event-driven, non-blocking I/O model that makes it lightweight and efficient. Node.js' package ecosystem, npm, is the largest ecosystem of open source libraries in the world.

1. In your terminal type, `brew install node`.

2. To verify your installation, type `node -v` and `npm -v`. 

   Approximate expected results: `v4.1.x` and `2.14.x` respectively.

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

Tmux lets you switch easily between several programs in one terminal, detach them (they keep running in the background) and reattach them to a different terminal.

1. In your terminal type, `brew install tmux`.

2. To verify your installation, type `brew list | grep "tmux"`. You should see `tmux` as an output. 

## Sublime Text 2 

Sublime Text is a sophisticated text editor for code, markup and prose. You'll love the slick user interface, extraordinary features and amazing performance.

1. Download Sublime Text 2 [here](http://www.sublimetext.com/2). 

2. Open Sublime and access the console, ``ctrl+`` or `View > Show Console`. 

3. Paste the following into the Sublime console:

~~~
import urllib.request,os,hashlib; h = '2915d1851351e5ee549c20394736b442' + '8bc59f460fa1548d1514676163dafc88'; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) ); by = urllib.request.urlopen( 'http://packagecontrol.io/' + pf.replace(' ', '%20')).read(); dh = hashlib.sha256(by).hexdigest(); print('Error validating download (got %s instead of %s), please try manual install' % (dh, h)) if dh != h else open(os.path.join( ipp, pf), 'wb' ).write(by)
~~~

4. Verify you can install packages by accessing the command palette, <kbd>⌘</kbd>+<kbd>Shift</kbd>+<kbd>P</kbd>. In the Command Palette, you should now be able to use `Package Control: Install Package`. 

*You can find a list of available packages [here](https://packagecontrol.io/browse).*

