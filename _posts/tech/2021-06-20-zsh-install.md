---
layout: post
title: oh-my-zsh环境安装
category: 技术
tags: [工具, oh-my-zsh]
keywords: oh-my-zsh
---

第一步，shell切换成zsh

查看当前shell `echo $SHELL`

查看有哪些shell `cat /etc/shells`

设置成zsh `chsh -s /bin/zsh`

第二步，安装oh-my-zsh

`git clone git://github.com/robbyrussell/oh-my-zsh.git ~/.oh-my-zsh`

执行完后.zshrc文件会被覆盖掉，目前我使用的配置如下:

	# Path to your oh-my-zsh installation.
	export ZSH="/Users/hupeng/.oh-my-zsh"
	
	# See https://github.com/ohmyzsh/ohmyzsh/wiki/Themes
	# 主题选择
	ZSH_THEME="robbyrussell"
	
	# 安装插件
	plugins=(git osx autojump zsh-autosuggestions zsh-syntax-highlighting)
	## autojump配置，autojump需要手动安装
	[[ -s /Users/hupeng/.autojump/etc/profile.d/autojump.sh ]] && source /Users/hupeng/.autojump/etc/profile.d/autojump.sh
	
	source $ZSH/oh-my-zsh.sh
	
	# User configuration
	
	# export MANPATH="/usr/local/man:$MANPATH"
	
	# You may need to manually set your language environment
	# export LANG=en_US.UTF-8
	
	# add by enderhu
	alias ll="ls -lha"

第三步，插件安装

	# 自动提示插件
	git clone git://github.com/zsh-users/zsh-autosuggestions $ZSH_CUSTOM/plugins/zsh-autosuggestions
	# 语法高亮插件
	git clone git://github.com/zsh-users/zsh-syntax-highlighting $ZSH_CUSTOM/plugins/zsh-syntax-highlighting
	
autojump插件安装有点麻烦，需要手动下载 https://github.com/wting/autojump/releases/tag/release-v22.5.3 并安装，然后.zshrc配置文件中加入一行对应配置，如上
