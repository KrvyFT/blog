---
title: Fish Shell
date: 2024-01-29 01:35:15
tags: Fish
categories: 笔记
---
# 笔记

Fish 脚本的 `shebang` 行，在文件开头加入这一行。

```sh
#!/usr/bin/fish
```

## 变量

Fish 中所有的变量都是字符串类型，不仅可以直接赋予字面值 还可以将一条 Shell 命令的输出存入变量中。

```sh
set VAL "Some value"
set CMD_VALUE = (uptime)
```

### 作用域

变量基本上有三种作用域

* 本地变量 local：只存在于函数内，使用 `-l` 设置。

* 全局变量 global：可用于同一 Shell 中的所有函，使用 `-g` 设置。

* 通用变量 universal：用于系统环境变量，在 Shell 重启后任然存在，使用 `-U/-gx` 设置。

### 列表

Fish 中所有变量都可以是列表

```sh
set LIST "One" "Two" "Three"
echo $LIST[1] # "One"
echo $LIST[2] # "Two"
echo $LIST[3] # "Three"
echo $LIST[-1] # "Three"
echo $LIST[1..3] # "One" "Two" "Three"
```

## 循环

### for

大致与 Rust 一样。

```sh
for VAL in &LIST
  echo $VAL
end
```

### while

```sh
for VAL in (seq 5)
    echo $VAL
end

set V 1
while test $V -lt 5
    echo $V
    set V (math $V + 1)
end
```

## 条件语句

if 的关键为 使用 test 语句对表达式求布尔值，下面是几个例子。

### 常规的条件检查

```sh
if test (count $argv) -lt 2
    echo "Usage: my-script <arg1> <arg2>"
    echo "Eg: <arg1> can be 'foo', <arg2> can be 'bar'"
else
    echo "Do something with $arg1 $arg2"
end
```

### 变量比较

```sh
set V foo

if test $V = foo
    echo bar
else
    echo "Err()"
end
```

### 检查文件是否存在

```sh
if test -e "file.sh"
    echo "somefile exists"
end
```

### 检查文件夹是否存在

```sh
if test -d "somefolder"
  echo "somefolder exists"
end
```

### 通配符情况

```sh
set file /home/krvy/Downloads/Telegram\ Desktop/*.py

if test (count file) -gt 0
    mv $file /home/krvy/Downloads/
    echo "Move $file to Download dir"
else
    echo "No python file found in dir"
end
```
