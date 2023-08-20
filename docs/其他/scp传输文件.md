## 案例

从一个实例解析scp的使用：

```bash
scp -r ~/tmp root@xxx.xxx.xxx.xxx:/home
```

`scp`：通过scp传输文件。

`-r`：传输文件夹。

`~/tmp`：传输的文件夹路径为`~`，名称是`tmp`。

`root@xxx.xxx.xxx.xxx`：root是指服务器的用户，@后面的是服务器的公网ip。

`:/home`：目的地址是`/home`。