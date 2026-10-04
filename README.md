# My Neovim Config

---

## Requirements
This Project require [entangled-cli](https://github.com/entangled/entangled.py).
You can install with:
```bash
uv tool install entangled-cli.
```

## Usage

```bash
git clone https://github.com/ShaoGuGong/nvim-config.git ~/.config/nvim
entangled tangle
```


## Initialization

load config files in 'init.lua':

```lua
--| file: init.lua
require("config.autocmds")
require("config.keymaps")
require("config.options")
```

download plugins in "./config/plugins.lua" and load them in 'init.lua':

```lua
--| file: init.lua
local plugins = require("config.plugins")
vim.pack.add(plugins)

local plugins_dir = vim.fs.joinpath(vim.fn.stdpath("config"), "lua", "plugins")
for file_name, type in vim.fs.dir(plugins_dir, { follow = true }) do
    if (type == "file" or type == "link") and file_name:match("%.lua$") and file_name ~= "init.lua" then
        local module = file_name:gsub("%.lua$", "")
        require("plugins." .. module)
    end
end
```

## Configuration
All Configuration on [config file](./docs/config.md).

## Plugins List

### Plugins

```lua
--| id: plugin-list
local plugins = {
    { src = gh("chrisgrieser/nvim-origami"), name = "nvim-origami" },            -- fold
    { src = gh("mrcjkb/rustaceanvim"), version = vim.version.range("^9") },      -- for rust development
    { src = gh("kylechui/nvim-surround"), version = vim.version.range("^4") },   -- surround
    { src = gh("saghen/blink.cmp"), version = vim.version.range("1.*") },        -- autocomplete
    gh("zbirenbaum/copilot.lua"),                      -- copilot autocomplete
    gh("fang2hou/blink-copilot"),                      -- blink for copilot
    gh("nvim-tree/nvim-web-devicons"),                 -- icon
    gh("nvim-lualine/lualine.nvim"),                   -- lualine
    gh("stevearc/oil.nvim"),                           -- file explorer
    gh("wakatime/vim-wakatime"),                       -- record coding time
    gh("folke/which-key.nvim"),                        -- show keymaps
    gh("neovim/nvim-lspconfig"),                       -- lsp config
    gh("mason-org/mason.nvim"),                        -- mason: for lsp
    gh("williamboman/mason-lspconfig.nvim"),           --mason-lspconfig
    gh("stevearc/conform.nvim"),                       -- formater
    gh("windwp/nvim-autopairs"),                       -- autopairs
    gh("lewis6991/gitsigns.nvim"),                     -- git signs
    gh("alexpasmantier/tv.nvim"),                      -- television
    gh("folke/noice.nvim"),                            -- notification
    gh("MunifTanjim/nui.nvim"),                        -- dependencies for noice.nvim
    gh("folke/flash.nvim"),                            -- quick jump
    gh("nvim-treesitter/nvim-treesitter"),             -- treesitter
    gh("HiPhish/rainbow-delimiters.nvim"),             -- colorful parentheses
    gh("stevearc/aerial.nvim"),                        -- code outline
    gh("rachartier/tiny-inline-diagnostic.nvim"),      -- inline diagnostic
    gh("andweeb/presence.nvim"),                       -- detect discord presence
    gh("karb94/neoscroll.nvim"),                       -- smooth scroll
    gh("rafamadriz/friendly-snippets"),                -- snippets
    gh("nvim-lua/plenary.nvim"),                       -- dependencies for telescope.nvim
    gh("nvim-telescope/telescope.nvim"),               -- telescope
    gh("m00qek/baleia.nvim"),                          -- dependencies for compile-mode
    gh("ej-shafran/compile-mode.nvim"),                -- emacs compile-mode for neovim
    gh("chomosuke/typst-preview.nvim"),                -- typst preview
    gh("folke/edgy.nvim"),                             -- edgy
    gh("lukas-reineke/indent-blankline.nvim"),         -- indent guide
    gh("MeanderingProgrammer/render-markdown.nvim"),   -- render markdown
}
```

### Color Scheme

```lua
--| id: colorscheme-list
local colorschemes = {
    { src = gh("neanias/everforest-nvim"), name = "everforest" },
    { src = cb("evergarden/nvim.git"), name = "evergarden" },
    { src = gh("ellisonleao/gruvbox.nvim"), name = "gruvbox" },
    { src = gh("rebelot/kanagawa.nvim"), name = "kanagawa" },
    { src = gh("shaunsingh/nord.nvim"), name = "nord" },
    { src = gh("AlexvZyl/nordic.nvim"), name = "nordic" },
    { src = gh("Mofiqul/vscode.nvim"), name = "vscode" },
}
```

---

## License 

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

