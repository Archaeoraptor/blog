---
title: 记一次git错误：“error object file .git/objects/.... is empty”
date: 2024-01-08 08:53:04
abbrlink: 'git-error-object-file-is-empty'
tags:
- git
---
git又坏了。系统又卡死了，tty都调不出来，强按关机键重启后出现的。应该是git操作中途强退导致的数据损毁。见[How can I fix the Git error "object file ... is empty"?](https://stackoverflow.com/questions/11706215/how-can-i-fix-the-git-error-object-file-is-empty) 
<!-- more -->

```bash
$git status
error: object file .git/objects/90/45d3aeb3539443f6b0a904935e9eb559fa5ef0 is empty
error: object file .git/objects/90/45d3aeb3539443f6b0a904935e9eb559fa5ef0 is empty
fatal: bad object HEAD
```

修复倒也简单，之前传了一份到github上了。把本地的repo删了重新clone一遍就好了。

但是这些断点损毁的问题还挺好玩的，尝试一下能不能手动修复。先执行fsck检查一下，出现了一堆error。错误主要有

```bash
~/codes/cg/ray_tracing ❯ git fsck --full                                 ✘ INT
error: inflate: data stream error (unknown compression method)
error: unable to unpack header of .git/objects/30/c28c65105992b8bf597f0388378c4724e163a5
error: 30c28c65105992b8bf597f0388378c4724e163a5: object corrupt or missing: .git/objects/30/c28c65105992b8bf597f0388378c4724e163a5
error: object file .git/objects/35/bfb29f7bf7d9940faa7b6859a1c3df91cca116 is empty
error: unable to mmap .git/objects/35/bfb29f7bf7d9940faa7b6859a1c3df91cca116: No such file or directory
error: 35bfb29f7bf7d9940faa7b6859a1c3df91cca116: object corrupt or missing: .git/objects/35/bfb29f7bf7d9940faa7b6859a1c3df91cca116
error: object file .git/objects/36/c81d09506146f59c67b572ad816bbecb60db1c is empty
error: unable to mmap .git/objects/36/c81d09506146f59c67b572ad816bbecb60db1c: No such file or directory
error: 36c81d09506146f59c67b572ad816bbecb60db1c: object corrupt or missing: .git/objects/36/c81d09506146f59c67b572ad816bbecb60db1c
error: object file .git/objects/56/1f5c8ca81e1ecb6f98c614b3d5f31309c1b037 is empty
error: unable to mmap .git/objects/56/1f5c8ca81e1ecb6f98c614b3d5f31309c1b037: No such file or directory
error: 561f5c8ca81e1ecb6f98c614b3d5f31309c1b037: object corrupt or missing: .git/objects/56/1f5c8ca81e1ecb6f98c614b3d5f31309c1b037
error: object file .git/objects/86/74f05d2355cb5ca9b32cdc2c2939f2fcc921d4 is empty
error: unable to mmap .git/objects/86/74f05d2355cb5ca9b32cdc2c2939f2fcc921d4: No such file or directory
error: 8674f05d2355cb5ca9b32cdc2c2939f2fcc921d4: object corrupt or missing: .git/objects/86/74f05d2355cb5ca9b32cdc2c2939f2fcc921d4
error: object file .git/objects/90/45d3aeb3539443f6b0a904935e9eb559fa5ef0 is empty
error: unable to mmap .git/objects/90/45d3aeb3539443f6b0a904935e9eb559fa5ef0: No such file or directory
error: 9045d3aeb3539443f6b0a904935e9eb559fa5ef0: object corrupt or missing: .git/objects/90/45d3aeb3539443f6b0a904935e9eb559fa5ef0
error: inflate: data stream error (unknown compression method)
error: unable to unpack header of .git/objects/c1/d4bc9d219d8b729d9b7e8a865f8972beac3014
error: c1d4bc9d219d8b729d9b7e8a865f8972beac3014: object corrupt or missing: .git/objects/c1/d4bc9d219d8b729d9b7e8a865f8972beac3014
error: object file .git/objects/cb/4fbbc9dcff805f550461cf27a860c1f884b77c is empty
error: unable to mmap .git/objects/cb/4fbbc9dcff805f550461cf27a860c1f884b77c: No such file or directory
error: cb4fbbc9dcff805f550461cf27a860c1f884b77c: object corrupt or missing: .git/objects/cb/4fbbc9dcff805f550461cf27a860c1f884b77c
error: object file .git/objects/d2/f1a78796402cb9858f66c7fe0d3dd90855fac1 is empty
error: unable to mmap .git/objects/d2/f1a78796402cb9858f66c7fe0d3dd90855fac1: No such file or directory
error: d2f1a78796402cb9858f66c7fe0d3dd90855fac1: object corrupt or missing: .git/objects/d2/f1a78796402cb9858f66c7fe0d3dd90855fac1
error: object file .git/objects/da/3cd6e317e0deb80fd7beda238d7baae23d2e60 is empty
error: unable to mmap .git/objects/da/3cd6e317e0deb80fd7beda238d7baae23d2e60: No such file or directory
error: da3cd6e317e0deb80fd7beda238d7baae23d2e60: object corrupt or missing: .git/objects/da/3cd6e317e0deb80fd7beda238d7baae23d2e60
error: object file .git/objects/e3/d3c4a99ac521fd488886206c5677f1b7f526b7 is empty
error: unable to mmap .git/objects/e3/d3c4a99ac521fd488886206c5677f1b7f526b7: No such file or directory
error: e3d3c4a99ac521fd488886206c5677f1b7f526b7: object corrupt or missing: .git/objects/e3/d3c4a99ac521fd488886206c5677f1b7f526b7
error: object file .git/objects/f3/af97d3545900a65d722ee8ba661bad4e632884 is empty
error: unable to mmap .git/objects/f3/af97d3545900a65d722ee8ba661bad4e632884: No such file or directory
error: f3af97d3545900a65d722ee8ba661bad4e632884: object corrupt or missing: .git/objects/f3/af97d3545900a65d722ee8ba661bad4e632884
error: object file .git/objects/fb/2dc1179dc5274f930b0c0a030c38cbb1c014fd is empty
error: unable to mmap .git/objects/fb/2dc1179dc5274f930b0c0a030c38cbb1c014fd: No such file or directory
error: fb2dc1179dc5274f930b0c0a030c38cbb1c014fd: object corrupt or missing: .git/objects/fb/2dc1179dc5274f930b0c0a030c38cbb1c014fd
Checking object directories: 100% (256/256), done.
error: object file .git/objects/90/45d3aeb3539443f6b0a904935e9eb559fa5ef0 is empty
error: object file .git/objects/90/45d3aeb3539443f6b0a904935e9eb559fa5ef0 is empty
error: refs/heads/master: invalid sha1 pointer 9045d3aeb3539443f6b0a904935e9eb559fa5ef0
error: object file .git/objects/90/45d3aeb3539443f6b0a904935e9eb559fa5ef0 is empty
error: object file .git/objects/90/45d3aeb3539443f6b0a904935e9eb559fa5ef0 is empty
error: refs/remotes/origin/master: invalid sha1 pointer 9045d3aeb3539443f6b0a904935e9eb559fa5ef0
error: object file .git/objects/90/45d3aeb3539443f6b0a904935e9eb559fa5ef0 is empty
error: object file .git/objects/90/45d3aeb3539443f6b0a904935e9eb559fa5ef0 is empty
error: HEAD: invalid sha1 pointer 9045d3aeb3539443f6b0a904935e9eb559fa5ef0
error: refs/heads/master: invalid reflog entry 9045d3aeb3539443f6b0a904935e9eb559fa5ef0
error: HEAD: invalid reflog entry 9045d3aeb3539443f6b0a904935e9eb559fa5ef0
error: object file .git/objects/36/c81d09506146f59c67b572ad816bbecb60db1c is empty
error: object file .git/objects/36/c81d09506146f59c67b572ad816bbecb60db1c is empty
error: 36c81d09506146f59c67b572ad816bbecb60db1c: invalid sha1 pointer in cache-tree of .git/index
missing blob 30c28c65105992b8bf597f0388378c4724e163a5
missing blob 35bfb29f7bf7d9940faa7b6859a1c3df91cca116
dangling tree 4b825dc642cb6eb9a060e54bf8d69288fbee4904
missing blob 561f5c8ca81e1ecb6f98c614b3d5f31309c1b037
missing blob 8674f05d2355cb5ca9b32cdc2c2939f2fcc921d4
missing blob c1d4bc9d219d8b729d9b7e8a865f8972beac3014
missing blob cb4fbbc9dcff805f550461cf27a860c1f884b77c
missing blob d2f1a78796402cb9858f66c7fe0d3dd90855fac1
missing blob da3cd6e317e0deb80fd7beda238d7baae23d2e60
missing blob e3d3c4a99ac521fd488886206c5677f1b7f526b7
missing blob fb2dc1179dc5274f930b0c0a030c38cbb1c014fd
```

object corrupt or missing 和 empty object的问题可以直接删除损坏的object，然后接着修下面的报错

```bash
~/codes/cg master ✚ ? ❯ git fsck --full                                          
Checking object directories: 100% (256/256), done.
broken link from    tree f3af97d3545900a65d722ee8ba661bad4e632884
              to    blob c1d4bc9d219d8b729d9b7e8a865f8972beac3014
dangling tree 4b825dc642cb6eb9a060e54bf8d69288fbee4904
missing blob c1d4bc9d219d8b729d9b7e8a865f8972beac3014
```

broken link的问题可以通过删除git的index后重新生成解决。

```bash
rm .git/index
git reset
```

此时git能正常使用了，但还是存在dangling tree 和 missing blob，dangling tree不是错误，见[git: dangling blobs](https://stackoverflow.com/questions/9955713/git-dangling-blobs)  

missing blob 的问题可以找到有问题的blob对应的文件，然后重新生成。

```bash
~/codes/cg master ● ⍟1 ❯ git ls-tree -r HEAD | grep c1d4bc9d219d8b729d9b7e8a865f8972beac3014
100644 blob c1d4bc9d219d8b729d9b7e8a865f8972beac3014    ray_tracing/CMakeLists.txt
git hash-object -w ray_tracing/CMakeLists.txt
```

或者用`git reflog`直接删了也可以

```bash
git reflog expire --expire=now --all
```

修复完成，收工。

```bash
~/codes/cg master ❯ git fsck --full                              
Checking object directories: 100% (256/256), done.
Checking objects: 100% (128/128), done.
```