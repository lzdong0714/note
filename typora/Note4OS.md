## Note4OS

### 南大b站OS course follow

``` http
http://jyywiki.cn/OS/2022/
```



### 工具安装与使用

#### vim

``` shell
# 直接输入vimtutor 查看并编写
$ vimtutor 

# command model:
delete word: dw
delete line: dd
delete begin to line end : d$

# search:
search word: 
```



tmux

``` shell
# 分屏
$ yum install -y tmux

# 使用
$ tmux

# cut by left and right
$ ctrl + b then "

# cut by up and down 
$ ctrl + b the %

# jump screen
$ ctrl + b then o

# exit screen 
$ ctrl + b then x
```



### gcc

``` shell
# download file by liunx
wget ${url}

# compile
$ gcc -o ${tarFile} ${srcFile.c}

# run 
$ ${tarFile}

```



### tldr

``` shell
# 获取shell命令的使用帮助

# 安装npm 然后执行
$ npm install -g tldr

## 使用，帮助ls 命令
$ tldr ls

 ls

  List directory contents.
  More information: https://www.gnu.org/software/coreutils/ls.

  - List files one per line:
    ls -1

  - List all files, including hidden files:
    ls -a

  - List all files, with trailing / added to directory names:
    ls -F

  - Long format list (permissions, ownership, size, and modification date) of all files:
    ls -la

  - Long format list with size displayed using human-readable units (KiB, MiB, GiB):
    ls -lh

  - Long format list sorted by size (descending):
    ls -lS

  - Long format list of all files, sorted by modification date (oldest first):
    ls -ltr

  - Only list directories:
    ls -d */
```







