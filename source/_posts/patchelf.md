---
title: 使用patchelf更改可执行文件的动态库和glibc版本
date: 2024-02-04 13:57:25
tags:
- patchelf
abbrlink: 'patchelf'
---
有时有一些比较无奈的场景，比如glibc版本太低了跑不起来。又或者动态库找的路径或者版本不对。这个时候我们就可以利用patchelf修改，以免重新编译。
<!-- more -->

## patchelf

patchelf是一个很好用的工具，可以用来编辑rpath，也可以用来指定glibc

```bash
yay -S patchelf
```

然后会下载到libs目录下面。

```bash
~/codes/blog master !1 ❯ patchelf --help                                                                                            7s
syntax: patchelf
  [--set-interpreter FILENAME] 指定ld
  [--page-size SIZE] 指定页大小
  [--print-interpreter]
  [--print-os-abi]              Prints 'EI_OSABI' field of ELF header
  [--set-os-abi ABI]            Sets 'EI_OSABI' field of ELF header to ABI.
  [--print-soname]              Prints 'DT_SONAME' entry of .dynamic section. Raises an error if DT_SONAME doesn't exist
  [--set-soname SONAME]         Sets 'DT_SONAME' entry to SONAME. 设置DT_SONAME
  [--set-rpath RPATH]   指定rpath
  [--add-rpath RPATH]
  [--remove-rpath]
  [--shrink-rpath]
  [--allowed-rpath-prefixes PREFIXES]           With '--shrink-rpath', reject rpath entries not starting with the allowed prefix
  [--print-rpath]
  [--force-rpath]
  [--add-needed LIBRARY] 
  [--remove-needed LIBRARY]
  [--replace-needed LIBRARY NEW_LIBRARY]
  [--print-needed]
  [--no-default-lib]
  [--no-sort]           Do not sort program+section headers; useful for debugging patchelf.
  [--clear-symbol-version SYMBOL]
  [--add-debug-tag]
  [--print-execstack]           Prints whether the object requests an executable stack
  [--clear-execstack]
  [--set-execstack]
  [--rename-dynamic-symbols NAME_MAP_FILE]      Renames dynamic symbols. The map file should contain two symbols (old_name new_name) per line
  [--output FILE]
  [--debug]
  [--version]
  FILENAME...
```

需要patch的glibc和动态库可以下载glibc-all-in-one，或者从aur装

```bash
git clone https://github.com/matrix1001/glibc-all-in-one
python update-list
cat list
# 下载对应版本的链接器
# ubuntu18可能需要下载zstd
# apt install zstd
./download 2.38-3ubuntu1_amd64
```

## 例子

例子：vscode 1.86的remote要求glibc 2.28及以上，于是在各种祖传老服务器就连不上了。  
vscode这帮人乐死我了，他们甚至不愿意在breaking changes之前发个警告，不愧是win10更新蓝屏搞到怨声载道的巨硬。  

要知道一周之前就有人发issue讨论这个问题：

<https://github.com/microsoft/vscode/issues/203967> 结果官方给出的方案就是手动下载一个portable版本，用老版本

<https://github.com/microsoft/vscode/issues/203967> 于是更新之后果不其然被骂惨了

当然你也可以给祖传服务器升级glibc，不过glibc这么重要的依赖，祖宗之法不可变。当然是想办法改改vscode了，这个时候再次轮到patchelf

首先移除`.vscode-server`，然后用vscode remote重新连接一遍。

然后用patchelf修改vscode server需要的node，使用自己下载的glibc的链接器, 并替换rpath

```bash
patchelf --set-interpreter ./glibc-all-in-one/libs/2.38-3ubuntu1_amd64/ld-linux-x86-64.so.2 --set-rpath ./glibc-all-in-one/libs/2.38-3ubuntu1_amd64/:./mygcc/lib64 --force-rpath ~/.vscode-server/bin/05047486b6df5eb8d44b2ecd70ea3bdf775fd937/node
```




