# Linux特殊文件夹的作用

[toc]

## /bin，/usr/bin，

1. 存放指令，通常**所有用户**都可以用。

2. **后期安装**的指令存放在`usr/bin`中。

## /sbin，/usr/sbin

1. 存放指令，通常只有**超级用户**都可以用。
2. **后期安装**的指令存放在`usr/sbin`中。



## /usr/share  /usr/local/share

Any program or package which contains or requires data that doesn't need to be modified should store that data in `/usr/share` (or `/usr/local/share`, if installed locally).
