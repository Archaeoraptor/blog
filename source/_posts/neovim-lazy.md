---
title: 迁移到Neovim的新插件管理器lazy.nvim
date: 2023-09-01 10:16:48
abbrlink: "neovim-lazy"
tags:
- neovim
- lazy
---
很久没写博客了，工作安定下来终于可以继续摸鱼了。
<!-- more -->

Neovim这些插件的半衰期太短了，几个月不折腾就跟不上了。刚用Neovim的时候就听说过很火的Packer.nvim插件管理器，当时Neovim可以用vim的vim-plug当插件管理器，就一直懒得往packer迁移，一直拖到现在，结果我还没迁移packer就已经凉了，emmmm

## lazy.nvim安装

AURe有一个包，我没用，照着github上的官方文档

在`~/.config/nvim/init.lua`或者`~/.config/nvim/init.lua`中添加

```lua
-- 指定插件位置，不存在则clone到本地
local lazypath = vim.fn.stdpath("data") .. "/lazy/lazy.nvim"
if not vim.loop.fs_stat(lazypath) then
  vim.fn.system({
    "git",
    "clone",
    "--filter=blob:none",
    "https://github.com/folke/lazy.nvim.git",
    "--branch=stable", -- latest stable release
    lazypath,
  })
end
vim.opt.rtp:prepend(lazypath)

-- 照例require一下，如果同时使用vim-plug插件，记得把这个放在vim-plug后面
require("lazy").setup(plugins, opts)
```

然后打开nvim，在normal mode下`:checkhealth lazy`，能看到3个ok就行了

推荐将插件配置扔到单独的一个lua文件，将上面的require改为

```lua
require("lazy").setup("plugins")
```

然后添加插件，在`~/.config/nvim/lua/plugins.lua`中或者`~/.config/nvim/lua/plugins/*lua`中，比如im-select插件：

```lua
return {
    "keaising/im-select.nvim",
    config = function()
        require("im_select").setup({})
    end,
}
```

然后启动nvim，按照提示按i安装就行了。

之前从vim迁到neovim的时候，用的coc.nvim。今年coc的作者已经不怎么更新了，曾经在知乎和推上很活跃的作者也不怎么网上冲浪了（记得曾经是位emacs铁粉，ghhn网上热情极高）。

用了Neovim默认的lsp, 然后加了个美化插件，功能依然不如coc.nvim。


```lua
return {
        "neovim/nvim-lspconfig",
    },
```

```lua
return {
    'nvimdev/lspsaga.nvim',
    config = function()
        require('lspsaga').setup({})
    end,
    dependencies = {
        'nvim-treesitter/nvim-treesitter', -- optional
        'nvim-tree/nvim-web-devicons'     -- optional
    },
}
```

## 链接

<https://github.com/folke/lazy.nvim>
