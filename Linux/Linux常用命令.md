### Linux常用命令

#### 文件管理

```shell
# 文件查找相关的
whereis # 指令只能用于查找二进制文件、源代码文件和man手册页，一般文件的定位需使用locate命令。 
which # which指令会在环境变量$PATH设置的目录里查找符合条件的文件。
locate # 用于查找符合条件的文档, locate your_file_name（文件名也可以是个路径）-i参数是忽略大小写
find # 命令用来在指定目录下查找文件。任何位于参数之前的字符串都将被视为欲查找的目录名。

# 内容查找相关的

```

#### vim

```bash
# 进入插入模式
'i'
'o' 新建一行插入
# 跳到文本最后一行
'G' 
# 跳到文本第一行
'H' 
# 查找
'/'+要查找的字符
# 替换
':s/old/new' #替换当前行
':%s/old/new' #替换每一行的第一个符合条件的字符
':%s/old/new/g' #全局替换
':3,5s/old/new' #指定行替换
# 执行命令
':!'+命令 #例如 :!ifconfig
# 设置
# 显示行号 
:set nu
# 不显示行号
:set nonu
# 全局配置需要修改vimrc
# mac根路径下没有这个文件 copy到根路径
cp  /usr/share/vim/vimrc  ~/.vimrc
```



#### Grep

```bash
# 匹配行数据包含a或b或c的数据
grep -E 'a|b|c' filename
grep 'a\|b\|c' filename
# 匹配行数据包含a和b和c的数据
grep 'a' filename | grep 'b' | grep 'c' 

# 忽略大小写
grep -i
# 正则表达式
grep -E
```



#### find

```bash
# 在指定文件夹下查找指定文件
find path -name filename 
# 例如：在根目录查找文件名为broker.conf的路径
find / -name broker.conf

# 在指定文件夹下查找指定文件夹，如果不加-type会把文件和文件夹都输出
find path -name filename -type d
# 例如：在根目录查找文件夹名为store的路径
find / -name store -type d


```



#### netstat

```bash
# 查找端口占用
netstat -npl | grep 8080
```



#### 进程监控

```bash
top命令
top 查看当前系统进程信息
'P':按CPU排序
'b':高亮显示排序列
'Shift+>或<':切换排序列

# 查看进程信息
ps -ef | grep 'java'
```



#### Vim常见配置

```bash
"显示行号
set nu

"启动时隐去援助提示
set shortmess=atI

"语法高亮
syntax on

"不需要备份
set nobackup

set nocompatible

"没有保存或文件只读时弹出确认
set confirm

"鼠标可用
set mouse=a

"tab缩进
set tabstop=4
set shiftwidth=4
set expandtab
set smarttab

"文件自动检测外部更改
set autoread

"c文件自动缩进
set cindent

"自动对齐
set autoindent

"智能缩进
set smartindent

"高亮查找匹配
set hlsearch

"显示匹配
set showmatch

"显示标尺，就是在右下角显示光标位置
set ruler

"去除vi的一致性
set nocompatible

"设置键盘映射，通过空格设置折叠
nnoremap <space> @=((foldclosed(line('.')<0)?'zc':'zo'))<CR>
""""""""""""""""""""""""""""""""""""""""""""""
"不要闪烁
set novisualbell

"启动显示状态行
set laststatus=2

"浅色显示当前行
autocmd InsertLeave * se nocul

"用浅色高亮当前行
autocmd InsertEnter * se cul

"显示输入的命令
set showcmd

"被分割窗口之间显示空白
set fillchars=vert:/
set fillchars=stl:/
set fillchars=stlnc:/

" vundle 环境设置
filetype off
set rtp+=~/.vim/bundle/Vundle.vim
"vundle管理的插件列表必须位于 vundle#begin() 和 vundle#end() 之间
call vundle#begin()
Plugin 'VundleVim/Vundle.vim'
Plugin 'altercation/vim-colors-solarized'
Plugin 'tomasr/molokai'
Plugin 'vim-scripts/phd'
Plugin 'Lokaltog/vim-powerline'
Plugin 'octol/vim-cpp-enhanced-highlight'
Plugin 'Raimondi/delimitMate'
" 插件列表结束
call vundle#end()
filetype plugin indent on

" 配色方案
set background=dark
colorscheme torte
"colorscheme molokai
"colorscheme phd

" 禁止显示菜单和工具条
set guioptions-=m
set guioptions-=T

" 总是显示状态栏
set laststatus=2

" 禁止折行
set nowrap

" 设置状态栏主题风格
let g:Powerline_colorscheme='solarized256'

syntax keyword cppSTLtype initializer_list

" 基于缩进或语法进行代码折叠
"set foldmethod=indent
set foldmethod=syntax
" 启动 vim 时关闭折叠代码
set nofoldenable

"允许用退格键删除字符
set backspace=indent,eol,start

"编码设置
set encoding=utf-8

"共享剪切板
set clipboard=unnamed

" Don't write backup file if vim is being called by "crontab -e"
au BufWrite /private/tmp/crontab.* set nowritebackup nobackup
" Don't write backup file if vim is being called by "chpass"
au BufWrite /private/etc/pw.* set nowritebackup nobackup
```

