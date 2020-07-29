# Vim's Built In Package Manager

For many of you you are very happy with your own choice of Vim plugin manager

I've tried a few but ended up feeling like all the

    Plugin tpope/yetanother-plugin-i-need 

in my .vimrc file just feels like clutter. But how can we do without these
tools?

Well we could right our own plugins to suit our needs - but why waste time
reinventing the wheel when there are so many good plugins out there.

How could I manage the plugins then and still keep a tidy .vimrc


## Vim 8 built in package manager

With Vim 8.0 we saw that Vim quietly added a built in package manager of sorts.
It isn't the fully featured plugin we have all seen and loved.

It is little more than putting the plugin in a specific folder and adding the
vimscript command **packloadall** to your .vimrc

Most plugin managers require some sort of vimscript bracketing in the .vimrc
file letting the Plugin Manager know where to look. 

Vim's built-in plugin manager doesn't do this it just expects that all
auto-loaded plugins will be found in a specific directory, namely:

> ~.vimrc/pack/plugins/start

To install a plugin with Vim's built in plugin manager you just clone the GitHub
repo into that directory (creating a subdirectory named after the plugin).

Then you want to update your plugins and you've got a problem.

Some like George Ornbo (https://shapeshed.com/vim-packages/) have found a
solution using git submodules. 

But that involves installing packages by typing in stuff like this:

```bash
cd ~/dotfiles
git submodule init
git submodule add https://github.com/vim-airline/vim-airline.git
vim/pack/shapeshed/start/vim-airline
git add .gitmodules vim/pack/shapeshed/start/vim-airline
git commit
```

Updating packages like this:

```bash
git submodule update --remote --merge
git commit
```

And removing packages like this:
```bash
git submodule deinit vim/pack/shapeshed/start/vim-airline
git rm vim/pack/shapeshed/start/vim-airline
rm -Rf .git/modules/vim/pack/shapeshed/start/vim-airline
git commit
```

And for me if something needs a git submodule then it is time for me to look at
another solution.

I soon realised that to manage the plugins using Vim's built-in package manager meant a lot of git clones, git pulls and rm commands never mind the occasional :helptags ALL inside Vim.  

This meant to get a meaningful package manager working from Vim's basic
functions I would need several commands to install, list, update and remove
plugins.


## Bash or shell scripts

I then started thinking what if I wrote a shell script. Oh, actually I would
need to write several shell scripts. Then I would have to keep them in a GitHub
repo or something so I could install them quickly. Still not a simple solution.

I then remembered Bash functions. For those of you not familiar with Bash
functions - you can define them in any Bash shell script (even your .bashrc) and
they appear to the user as a builtin command of the shell or operating system. Although there 
isn't a binary or a shell script you can still auto complete using the tab key.


## Setting Up My Package Management System

Setting up the builtin Vim plugin management is simple. This script has the
following:

```bash
export DOTVIM=~/.vim
export VIMPLUGINS=${DOTVIM}/pack/plugins/start
mkdir -p $VIMPLUGINS        # make if not exist silent otherwise
```

The VIMPLUGINS variable is setup to point to the standard Vim plugin manager
directory for autostarting plugins. 

The script then runs a *mkdir -p* on that directory to ensure it exists

the only other thing you need to do is add the following command to the top of
your .vimrc file:

```vim
packloadall
```

This command autoloads everything inside your .vim/pack/plugins/start directory
tree.


## Installing a Vim plugin

To install a plugin you need to know either the GitHub repo OR the shortcut
reference that many Plugin managers use.

For example, Tim Pope has written the excellent vim-fugitive Git wrapper that
can be found at https://github.com/tpope/vim-fugitive

With my Bash functions you can simply type

```bash
$ viminstallplugin https://github.com/tpope/vim-fugitive
```

and the function will go ahead and clone the plugin into the correct directory.

but what if you saw somebody else's .vimrc typing the full URL can be a pain so
the function can also detect if you use the standard shortcut for the plugin: 

```bash
$ viminstallplugin tpope/vim-fugitive
```

(The installation script will also create a imreinstallplugins.sh shell script in
the plugins directory in case you need it that will clone each repository.)


## Updating installed Vim plugins

Most good plugin managers have a facility to realize when the installed plugin
is out of date and to update ti from the github repo.

Most of these do this by just running the command you use for installing a
plugin and off it goes.

I felt that this can be limiting in some situations and as I was writing these
bash functions I decided that if I had a bash function __vimupdateplugins__ I
could update the plugins inside vim using: 

```vim
:terminal
```
Then typing: 

```bash
$ vimupdateplugins
```

will systematically work it's way through each installed plugin and run a **git
pull** on each plugin directory updating the plugin frmo it's repo.

NB if you want you can write a quick shell wrapper to be run as a cron job and
you can wake up to a freshly updated set of plugins every day:

```bash
#!/usr/bin/env bash
source  ~/vim-builtin-packager/bashrc_additions
vimupdateplugins
```


## Listing installed Vim plugins 

To list the plugins you have installed simply run:

```bash
$ vimlistplugins
itchyny/calendar.vim
junegunn/goyo.vim
itchyny/lightline.vim
vifm/vifm.vim
jreybert/vimagit
tpope/vim-markdown
reedes/vim-pencil
vimwiki/vimwiki
```

The function lists all installed plugins (based on those listed in the
vimreinstallplugins.sh script that is updated every time a plugin is installed
or removed.


## Removing installed Vim plugins

Removing a plugin is simple. Yes you could go to the plugins directory (setup as
$VIMPLUGINS in the bash script) and rm -rf the directory. But why change
directory, just issue this command from anywhere:

```bash
$ vimremoveplugin vim-pencil
```

This function removes all traces from the plugin directory of the script (also
removing the git clone command from the vimreinstallplugins.sh script to keep
that in sync).


## Footnote

(By the way thanks to Tim Pope for all all his plugins - they truely make my day
better.)
