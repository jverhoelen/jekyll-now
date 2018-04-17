---
layout: post
title: Migrating from Fish Shell to Zsh
last_modified_at: "2016-12-26"
category: "Productivity"
---

I've been using Fish Shell for MacOS for about four years before I finally decided to switch to Zsh. Why? I was simply annoyed to Fishs incompatibilities with Bash commands. Remembering another syntax for setting and exporting environment variables and missing support for the ``&&`` operator were the top of the iceberg.

I started figuring out what needs to be done for the big change:

- Overtaking the Fish history to Zsh (essential for auto suggestion and `history | grep 'some forgotten cmd'`)
- Command auto suggestion and completion (very handy in Fish)
- Overtaking Fish functions you wrote yourself for convenience
- Themes. *Everyone* needs great themes. Especially with some great Git status indicating icons.

## Installing Zsh and oh-my-zsh

First, you need to [install Zsh](https://github.com/robbyrussell/oh-my-zsh/wiki/Installing-ZSH "How to install ZSH"){:target="_blank" rel="noopener noreferrer"} which is, for MacOS, done with a quick `brew install zsh zsh-completions`. Make it your users' default shell using `chsh -s $(which zsh)`.

If you want to use Zsh, you really also want [oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh "Oh-my-zsh as framework for Zsh"){:target="_blank" rel="noopener noreferrer"}. It's a community driven framework for plugins, themes configuration and upgrading capabilities which is very powerful. [Its installation](https://github.com/robbyrussell/oh-my-zsh#basic-installation "oh-my-zsh installation tutorial"){:target="_blank" rel="noopener noreferrer"} is also straightforward.

That's the basics.

## Migrating your history

Fish Shell writes its history to `~/.local/share/fish/fish_history` in a YAML format. Zsh uses `~/.zsh_history` with a CSV-like format. To automatically convert the Fish format to Zsh and append it, I wrote a small Node script. You can [find it on GitHub](https://github.com/jverhoelen/fish-history-to-zsh "fish-history-to-zsh transforms from fish to zsh format"){:target="_blank" rel="noopener noreferrer"} and use it out of the box. There you go!

## Fish-like auto suggestions

Auto suggestions is a cool Fish Shell feature – I don't want to miss it in Zsh. Fortunately there is already an oh-my-zsh plugin called [zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions#oh-my-zsh "zsh-autosuggestions contain autosuggestions for zsh"){:target="_blank" rel="noopener noreferrer"} with a quick installation. That was another easy step, huh?

## Migrating your functions

Fish lets you define custom commands in the appearance of functions in `.config/fish/functions`. For example I had `acp.fish` (add, commit, push for Git).

``` bash
function acp --description 'Add, commit and push'
    git add .
    git add --all
    git commit -m $argv
    git push
end
```

In Zsh you can just append them to your `~/.zshrc` which is the central configuration. It gets some changes now looks like that to work in Zsh:

``` bash
function acp() {
    git add .
    git add --all
    git commit -m "$1"
    git push
}
```


## Choosing a theme

We can just stay in `~/.zshrc` and search for the place where `ZSH_THEME` was set to `robbyrussell`, which is the default theme in oh-my-zsh. But there are [a lot more](https://github.com/robbyrussell/oh-my-zsh/wiki/Themes "List of oh-my-zsh themes"){:target="_blank" rel="noopener noreferrer"}, choose your favorite. I directly fell in love with [agnoster](https://github.com/robbyrussell/oh-my-zsh/wiki/themes#agnoster "Agnoster zsh theme"){:target="_blank" rel="noopener noreferrer"} (which has these fancy arrows) in combination with the Solarized theme for iTerm2.


## Conclusion

I'm totally happy with Zsh, oh-my-zsh and the agnoster theme. Migrating the Fish functions was pretty easy, old history is available and my shell is finally compatible with most of the important scripts one needs nowadays.

Maybe I could also help you with your decision or migration.
