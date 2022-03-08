# Steps For Beautify Your macOS Terminal

## 0. Brief Introduction

MacBook is one of the most popular laptops for programmers because macOS is the only UNIX certified operating system. For our programmer, `terminal.app` is our most frequently used app. You can work more efficiently if your terminal is more beautiful. When you first get a MacBook, you should beautify your terminal in time. Follow this step-by-step guide, you can get such a beautiful terminal.

![](https://pic.imgdb.cn/item/621e31df2ab3f51d91e8731f.png)

<center><i>System Default Terminal(previous)</i></center>

<img src="https://pic.imgdb.cn/item/621e32502ab3f51d91e95b46.png" style="zoom: 50%;" />

<center><i>Final Terminal🎉</i></center>

## 1. Prerequisites

When you first get a MacBook, you should install some tools and plugins. I list some of the most useful you should install below:

* Xcode command line tools, include **clang**、**python2**、**git**；
* Homebrew🍺;
* Node.js;
* Python3;

> ‼️WARNING‼️：For readers in China mainland, you'd better set a proxy that can get data from International Internet before the install process.
>
> ‼️     警告     ‼️：对于在中国大陆的读者，在开始安装进程前您最好设置好可以访问国际互联网的代理。

### 1.1 Xcode command line tools

Xcode command line tools is a set of useful tools from Apple. After you install this, you can develop C/C++ program and use git.

You should run this command in your terminal:

```shell
$ xcode-select --install
```

Or you can get Xcode in App Store, you just need to click 'Get'. However, it takes a long time because Xcode IDE is a nearly 20GB app.

### 1.2 Homebrew🍺

Homebrew is the most popular package manager in macOS. You can install many useful tools with homebrew. You can just run the command to install it:

```shell
$ /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

For more information, you can visit their official website: [https://brew.sh](https://brew.sh)

There are several commands during the install process, just follow it.

### 1.3 Node.js

Node.js is a powerful JS engine running locally. Many useful tools is written in node.js. `npm` is one of the most popular package manager.

You can go to their official website([https://nodejs.org/en/](https://nodejs.org/en/)) to install the `.pkg`, or use homebrew just like this:

```shell
$ brew install node
```

### 1.4 Python3

You can find that there is a build-in python in your new MacBook. But it's python2. There are many differences between python3 and python2. So you should install python3.

Familiar with Node.js, you can also install it from their official website or homebrew.

## 2. Oh My ZSH

Oh My Zsh is an open source, community-driven framework for managing your zsh configuration. If you want to change your terminal's theme, you must install it.

You can install it very easily, just run this command in your terminal:

```shell
$ sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

> ‼️WARNING‼️: If you have written something in `~/.zshrc`, please make a backup copy because oh my zsh will cover your configuration file.

## 3. iTerm2

iTerm2 is a popular terminal in macOS. It has many improvements. Many programmers use it as their default terminal.

So you should install iTerm2. I suggest you to use `.dmg` to install.

## 4. Download Fonts And Themes

The terminal's default font is not good-looking for us programmers for the reason that it is not a monospace font. Therefore, we should download a monospace font. Also, we should download a beautiful theme, I think that power level 10k is a good choice for you.

You can git clone the theme to your custom theme folder.

```shell
$ git clone https://github.com/romkatv/powerlevel10k.git $ZSH_CUSTOM/themes/powerlevel10k
```

`$ZSH_CUSTOM`  reffers to `~/.oh-my-zsh/custom`.

Because we should use a font that suitable for the theme, we should download these font to wherever you want. Go to this website [https://github.com/Falkor/dotfiles/blob/master/fonts/SourceCodePro%2BPowerline%2BAwesome%2BRegular.ttf](https://github.com/Falkor/dotfiles/blob/master/fonts/SourceCodePro%2BPowerline%2BAwesome%2BRegular.ttf) for download. You will get a `.ttf` file and you should open the font book app and drag the `.ttf` to the panel to add this font.

## 5. Set iTerm2

At this stage, you should set your iterm2 more beautiful by creating your own profile. Go to `Preference ->Profiles`, create your own profile (please give it a readble name and **do not delete default profile!**).

Then set your own profile as default and begin to set it more preferences.

Firstly, go to this website [https://raw.githubusercontent.com/utsavized/iterm2/develop/utsavized.itermcolors](https://raw.githubusercontent.com/utsavized/iterm2/develop/utsavized.itermcolors), you will see many html like strings. Copy it and paste it to a newly-created `filename.itermcolors`. Then go to color settings to set your `.itermcolors` as your colorset.

Then go to the text settings and set the font as your `SourcecodePro+...` font, and you can set your font size as options.

Finally go to the sessions settings and enable your status bar. You can see `Configution Status Bar` at the enable button's right, click it and add some widgets you want. As my opinion, it's necessary to add CPU, GPU and Network widgets because you want to know the current status of this computer when you want to excute some command.

## 6. Config p10k themes

The last main task you should complete is to let the p10k theme your default theme and config it when you first use it.

Just set theme on .zshrc: `ZSH_THEME="powerlevel10k/powerlevel10k"`, save your update and restart your iTerm2. You will enter the config process of p10k. Config it as you like, you will see many tips and previews.

Then your iTerm2 terminal is much more beautiful than before 🎉🎉🎉

## 7. After

The step 6 is not an end, when you use terminal with other software such as VS Code or IDEA, you will see that the apple logo '''' and other symbols become unknown. Don't worry! Just go to their Preferences and change the terminal font to your own `SourcecodePro...` and restart your Editors or IDEs. You will see that it just works(maybe not as beautiful as iTerm2😮‍💨).

## 8. Add Auto-Suggestion Plugin

The main advantage of oh-my-zsh is that there are many plugins available to improve your efficiency. I can't wait to recommend  `zsh-autosuggestion` plugin to you.

First you should install it to `${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions`. Run this command in your beautiful terminal.

```shell
$ git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
```

Then go to `.zshrc` file and find line around 79. You will see plugins setting(git plugin has been existed by default). Add `zsh-autosuggestions` and restart your terminal.

When you type command that you typed before, you will see the tips. You can just press right arrow ➡️ to complete this command quickly. Trust me, it's very helpful because it can save you much time.
