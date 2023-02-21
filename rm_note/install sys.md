# 装机教程

> 在装完Ubuntu20.04后，配置团队相关的东西的配置文档

Ubuntu20.04的安装在b站有许多教程，跟着来就好

## Linux的基本操作

- 打开终端：ctrl+alt+T（之后输入的命令都是输入到终端里面）
  - 终端就是输入命令的窗口，Windows的cmd一样
  - ==在终端内ctrl+shift+c才是复制，ctrl+shift+v粘贴==
- 基础命令
  - cd  进入一个文件夹
  - mkdir 创建一个文件夹
- Tab可以在输入命令时补全命令，可以增加敲命令的速度，也可以在忘记命令的时候起到提示的作用
- 按键盘上键可以重复上一条命令

## ros的安装

> ```
> wget http://fishros.com/install -O fishros && . fishros
> ```

这是运行这个脚本的命令，按照他提示就好,它会提醒你需要换源（换源是为了更快更好的安装一些东西），输入的全部为1121（==注意一定要安装ROS1，notice版本==）

## pip3

> 用于之后的下载用的

```c
sudo apt-get install python3-pip
```

## catkin_tools

> 用于之后运行代码

```
 sudo pip3 install -U catkin_tools
```

## rosmon

> 用来运行代码

```
sudo apt install ros-${ROS_DISTRO}-rosmon
```





## pigcha

> 用来翻墙的

http://101.34.95.10:8081/?platform=linux_x86_64&cur_version=1672215479

## Github

> 用于拉取团队代码和版本管理

下完pigcha之后翻墙，搜github创个号

## 团队代码

### 需要先了解一下Git，工作空间，ssh

- Git是一个工具用来管理代码的管理
- 工作空间是ROS下一种类似文件夹的东西，规定了里面的包含的最基本的东西
- ssh是用来安全连接的，用来登nuc和拉代码的

### 第一步：配置好ssh

[https://blog.csdn.net/weixin_48349367/article/details/120056192?spm=1001.2101.3001.6650.2&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7Edefault-2-120056192-blog-103099424.pc_relevant_vip_default&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7Edefault-2-120056192-blog-103099424.pc_relevant_vip_default&utm_relevant_index=3]

#### 第二步：创建一个工作空间

> http://wiki.ros.org/cn/ROS/Tutorials/InstallingandConfiguringROSEnvironment（官方教程，下面是具体操作的命令）

```
mkdir -p ~/rm_ws/src
cd ~/rm_ws/
catkin_make
```

#### 第三步：拉团队代码

###### 团队代码仓库

> https://github.com/rm-controls

拉代码的命令(先新开个终端)

```
cd rm_ws/src
git clone git@github.com:rm-controls/rm_control.git
git clone git@github.com:rm-controls/rm_controllers.git
git clone git@github.com:rm-controls/rm_manual.git
git clone git@github.com:rm-controls/rm_config.git
git clone git@github.com:rm-controls/rm_description.git
git clone git@github.com:rm-controls/rm_bringup.git
catkin built
```



## 配置.bashrc

```
vim .bashrc
```

==vim是一个编辑工具进去之后先按任意键就可以进入编辑模式，编辑完之后按ESC退出编辑模式，按 :wq（这是保存退出的意思）==

进去之后全删了复制粘贴下面的

```
# ~/.bashrc: executed by bash(1) for non-login shells.
# see /usr/share/doc/bash/examples/startup-files (in the package bash-doc)
# for examples

# If not running interactively, don't do anything
case $- in
    *i*) ;;
      *) return;;
esac

# don't put duplicate lines or lines starting with space in the history.
# See bash(1) for more options
HISTCONTROL=ignoreboth

# append to the history file, don't overwrite it
shopt -s histappend

# for setting history length see HISTSIZE and HISTFILESIZE in bash(1)
HISTSIZE=1000
HISTFILESIZE=2000

# check the window size after each command and, if necessary,
# update the values of LINES and COLUMNS.
shopt -s checkwinsize

# If set, the pattern "**" used in a pathname expansion context will
# match all files and zero or more directories and subdirectories.
#shopt -s globstar

# make less more friendly for non-text input files, see lesspipe(1)
[ -x /usr/bin/lesspipe ] && eval "$(SHELL=/bin/sh lesspipe)"

# set variable identifying the chroot you work in (used in the prompt below)
if [ -z "${debian_chroot:-}" ] && [ -r /etc/debian_chroot ]; then
    debian_chroot=$(cat /etc/debian_chroot)
fi

# set a fancy prompt (non-color, unless we know we "want" color)
case "$TERM" in
    xterm-color|*-256color) color_prompt=yes;;
esac

# uncomment for a colored prompt, if the terminal has the capability; turned
# off by default to not distract the user: the focus in a terminal window
# should be on the output of commands, not on the prompt
#force_color_prompt=yes

if [ -n "$force_color_prompt" ]; then
    if [ -x /usr/bin/tput ] && tput setaf 1 >&/dev/null; then
        # We have color support; assume it's compliant with Ecma-48
        # (ISO/IEC-6429). (Lack of such support is extremely rare, and such
        # a case would tend to support setf rather than setaf.)
        color_prompt=yes
    else
        color_prompt=
    fi
fi

if [ "$color_prompt" = yes ]; then
    PS1='${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '
else
    PS1='${debian_chroot:+($debian_chroot)}\u@\h:\w\$ '
fi
unset color_prompt force_color_prompt

# If this is an xterm set the title to user@host:dir
case "$TERM" in
xterm*|rxvt*)
    PS1="\[\e]0;${debian_chroot:+($debian_chroot)}\u@\h: \w\a\]$PS1"
    ;;
*)
    ;;
esac

# enable color support of ls and also add handy aliases
if [ -x /usr/bin/dircolors ]; then
    test -r ~/.dircolors && eval "$(dircolors -b ~/.dircolors)" || eval "$(dircolors -b)"
    alias ls='ls --color=auto'
    #alias dir='dir --color=auto'
    #alias vdir='vdir --color=auto'

    alias grep='grep --color=auto'
    alias fgrep='fgrep --color=auto'
    alias egrep='egrep --color=auto'
fi

# colored GCC warnings and errors
#export GCC_COLORS='error=01;31:warning=01;35:note=01;36:caret=01;32:locus=01:quote=01'

# some more ls aliases
alias ll='ls -alF'
alias la='ls -A'
alias l='ls -CF'

# Add an "alert" alias for long running commands.  Use like so:
#   sleep 10; alert
alias alert='notify-send --urgency=low -i "$([ $? = 0 ] && echo terminal || echo error)" "$(history|tail -n1|sed -e '\''s/^\s*[0-9]\+\s*//;s/[;&|]\s*alert$//'\'')"'

# Alias definitions.
# You may want to put all your additions into a separate file like
# ~/.bash_aliases, instead of adding them here directly.
# See /usr/share/doc/bash-doc/examples in the bash-doc package.

if [ -f ~/.bash_aliases ]; then
    . ~/.bash_aliases
fi

# enable programmable completion features (you don't need to enable
# this, if it's already enabled in /etc/bash.bashrc and /etc/profile
# sources /etc/bash.bashrc).
if ! shopt -oq posix; then
  if [ -f /usr/share/bash-completion/bash_completion ]; then
    . /usr/share/bash-completion/bash_completion
  elif [ -f /etc/bash_completion ]; then
    . /etc/bash_completion
  fi
fi
source /opt/ros/noetic/setup.bash
source ~/rm_ws/devel/setup.bash
export PATH=/home/lsy/snap/clion/198/bin:$PATH


export ROS_IP=`ifconfig | grep -Eo 'inet (addr:)?([0-9]*\.){3}[0-9]*' | grep -Eo '([0-9]*\.){3}[0-9]*' | grep -v '127.0.0.1' | grep -v '172.17.0.1'`

if  test -z "${ROS_IP}"
then
        export ROS_IP=`dig +short localhost`
fi

export ROS_MASTER_URI=http://192.168.2.220:11311
echo "ROS IP: ${ROS_IP}
ROS Master URI: ${ROS_MASTER_URI}"

alias setcan0='sudo ip link set can0 up type can bitrate 1000000'
alias setcan1='sudo ip link set can1 up type can bitrate 1000000'
alias setcan2='sudo ip link set can2 up type can bitrate 1000000'
alias setcan3='sudo ip link set can3 up type can bitrate 1000000'
alias engineer='ssh dynamicx@192.168.2.220'
alias balance='ssh dynamicx@192.168.1.110'
alias hero='ssh dynamicx@192.168.1.101'
alias standar4='ssh dynamicx@192.168.1.104'
alias standar5='ssh dynamicx@192.168.1.105'
alias standar3='ssh dynamicx@192.168.1.103'
alias sentry='ssh dynamicx@192.168.1.111'
export ROBOT_TYPE=engineer
alias engineer_client='rosrun actionlib_tools axclient.py /engineer_middleware/move_steps'
alias note='typora rm_note/rm_clion/rm_manual.md'
alias rqt_joint='rosrun rqt_joint_trajectory_controller rqt_joint_trajectory_controller'
export filepath="/etc/apt/"
alias load_ros='mon launch engineer_arm_config start.launch'
alias load_controller='. load.sh'
alias bringup="mon launch rm_bringup engineer.launch "
alias use_current="rosrun moveit_commander moveit_commander_cmdline.py"

export MVCAM_SDK_PATH=/opt/MVS

export MVCAM_COMMON_RUNENV=/opt/MVS/lib
export LD_LIBRARY_PATH=/opt/MVS/lib/64:/opt/MVS/lib/32:$LD_LIBRARY_PATH

```

## clion

> 用于看代码写代码

```
sudo snap install clion --classic
```

## 



> 以上基本配置就完成了！！！
