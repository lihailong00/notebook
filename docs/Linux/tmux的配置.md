[toc]



## tmux配置文件

```bash
set-option -g status-keys vi
setw -g mode-keys vi

setw -g monitor-activity on

# setw -g c0-change-trigger 10
# setw -g c0-change-interval 100

# setw -g c0-change-interval 50
# setw -g c0-change-trigger  75


set-window-option -g automatic-rename on
set-option -g set-titles on
set -g history-limit 100000

#set-window-option -g utf8 on

# set command prefix
set-option -g prefix C-a
unbind-key C-b
bind-key C-a send-prefix

bind h select-pane -L
bind j select-pane -D
bind k select-pane -U
bind l select-pane -R

bind -n M-Left select-pane -L
bind -n M-Right select-pane -R
bind -n M-Up select-pane -U
bind -n M-Down select-pane -D

bind < resize-pane -L 7
bind > resize-pane -R 7
bind - resize-pane -D 7
bind + resize-pane -U 7


bind-key -n M-l next-window
bind-key -n M-h previous-window



set -g status-interval 1
# status bar
set -g status-bg black
set -g status-fg blue


#set -g status-utf8 on
set -g status-justify centre
set -g status-bg default
set -g status-left " #[fg=green]#S@#H #[default]"
set -g status-left-length 20


# mouse support
# for tmux 2.1
# set -g mouse-utf8 on
set -g mouse on
#
# for previous version
#set -g mode-mouse on
#set -g mouse-resize-pane on
#set -g mouse-select-pane on
#set -g mouse-select-window on


#set -g status-right-length 25
set -g status-right "#[fg=green]%H:%M:%S #[fg=magenta]%a %m-%d #[default]"

# fix for tmux 1.9
bind '"' split-window -vc "#{pane_current_path}"
bind '%' split-window -hc "#{pane_current_path}"
bind 'c' new-window -c "#{pane_current_path}"

# run-shell "powerline-daemon -q"

# vim: ft=conf
```



## 安装方法以及配置路径

安装方法（centos）：

```bash
yum install tmux
```

配置路径：

在默认情况下，tmux 会在两个位置查找配置文件。首先查找 `/etc/tmux.conf` 作为系统配置，然后在当前用户的主目录下查找 `.tmux.conf` 文件（~/.tmux.conf 优先级更高）。如果这两个文件都不存在，tmux 就会使用默认配置。如果不需要使用系统配置，就只需要在主目录下创建一个新的配置文件即可（即 `~/.tmux.conf` 文件）。命令如下：

```bash
touch ~/.tmux.conf
```



备注：不同的tmux版本可能会修改部分信息。



## tmux使用教程

[tmux使用教程](https://www.acwing.com/file_system/file/content/whole/index/content/2855620/)



