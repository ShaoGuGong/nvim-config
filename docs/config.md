# Config

---

## Utils
```lua
--| file: lua/config/utils.lua
local utils = {}
utils.gh = function(repo_url)
    return "https://github.com/" .. repo_url
end

utils.cb = function(repo_url)
    return "https://codeberg.org/" .. repo_url
end
return utils
```

## Auto Commands

```lua
--| file: lua/config/autocmds.lua
<<autocmds-input-method>>
<<autocmds-indentation>>
```

### Control Input Method
This command will automatically switch the input method to English when leaving insert mode.

**only works on macOS.**
require  [im-select](https://github.com/daipeihust/im-select).

```lua
--| id: autocmds-input-method
local input_method_group  = vim.api.nvim_create_augroup("InputMethodToggle", { clear = true })
vim.api.nvim_create_autocmd("InsertLeave", {
    group = input_method_grooup,
    pattern = "*",
    callback = function()
        local target_im = "com.apple.keylayout.ABC"
        local current_im = vim.fn.system("im-select"):gsub("%s+", "")
        if current_im ~= target_im then
            vim.uv.spawn("im-select", { args = { target_im } }, function() end)
        end
    end,
})
```

### Control Indentation

```lua
--| id: autocmds-indentation
local dev_group = vim.api.nvim_create_augroup("DevSetting", { clear = true })
local c_dev_events = {
    "BufRead",
    "BufNewFile",
}
vim.api.nvim_create_autocmd(c_dev_events, {
    group = c_dev_group,
    pattern = { "*.c", "*.h", "*.cpp", "*.hpp", "*.tex", "*.typ", "*.lua" },
    callback = function()
        vim.opt_local.tabstop = 2
        vim.opt_local.shiftwidth = 2
    end,
})
```

## Keymaps

```lua
--| file: lua/config/keymaps.lua
vim.g.mapleader = " "
vim.g.localleader = "\\"

vim.keymap.set("n", "<C-S-j>", "i<enter><ESC>", { noremap = true, silent = true })
vim.keymap.set("t", "<ESC>", "<C-\\><C-n>", { noremap = true, silent = true })
```

## Options

```lua
--| file: lua/config/options.lua
vim.opt.guicursor = "a:block-blinkon500-blinkoff500-blinkwait500,i-ci:ver25-Cursor/lCursor,r-cr:hor20-Cursor,o:hor50"

vim.opt.cursorline = true
-- vim.o.colorcolumn = "80"
vim.opt.hidden = true
vim.opt.expandtab = true
vim.opt.tabstop = 4
vim.opt.shiftwidth = 4
vim.opt.softtabstop = 4
vim.opt.swapfile = false

vim.opt.textwidth = 120
vim.opt.wrap = false
vim.opt.linebreak = true
vim.opt.number = true
vim.opt.relativenumber = true
vim.opt.encoding = "utf8"
vim.opt.autoread = true

vim.opt.list = true
vim.opt.lcs = {
    tab = "󰌒 ",
    space = "·",
    -- trail = "󰥓",
    nbsp = "+",
}

vim.opt.termguicolors = true
vim.opt.clipboard = "unnamedplus"
vim.cmd([[au BufReadPost * if line("'\"") > 1 && line("'\"") <= line("$") | exe "normal! g'\"" | endif]])
vim.lsp.inlay_hint.enable(true)
vim.g.pyindent_open_paren = "shiftwidth()"
vim.g.pyindent_nested_paren = "shiftwidth()"
```
