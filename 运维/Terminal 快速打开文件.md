### Terminal 快速打开文件

#### typora配置

> ```bash
> # 这种方式是通用的
> # 在zshrc中加入下面配置，并通过source使其生效
> alias ty="open -a typora"
> 
> # 使用
> 在terminal中输入‘ty 文件名’，即可打开并编辑指定文件
> 在terminal中输入‘ty .’，即可打开整个文件夹并新建一个新的文件
> ```



#### VSCode配置

> 1. 打开 VSCode
>
> 2. 打开控制面板(`⇧⌘P`), 输入 `shell command`, 在提示里看到  `Shell Command: Install 'code' command in PATH`, 运行它就可以了。
>
>    ![img](https://tva1.sinaimg.cn/large/e6c9d24ely1h0nnijufe3j20h602s0su.jpg)
>
>    
>
>    
>
> 使用
>
> 输入code 文件名或者code .



 

#### Sublime配置

```bash
# 使用Sublime提供的命令行工具subl并创建软链接即可

sudo ln -s /Applications/Sublime\ Text.app/Contents/SharedSupport/bin/subl /usr/local/bin/subl

# 使用
输入subl 文件名 或者 subl .
```



