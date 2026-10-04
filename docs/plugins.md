# Plugins

---

## Download Plugins use vim.pack.add

```lua
--| id: repo_url
-- helper function to get repo url
local gh = function(repo_url)
    return "https://github.com/" .. repo_url
end

local cb = function(repo_url)
    return "https://codeberg.org/" .. repo_url
end
```

```lua
--| file: lua/config/plugins.lua
<<repo_url>>
<<colorscheme-list>>
<<plugin-list>>

vim.list_extend(plugins, colorschemes)
return plugins
```

---

## Plugin Configuration

### aerial

```lua
--| file: lua/plugins/aerial.lua
require("aerial").setup()
local wk = require("which-key")
wk.add({
    { "<leader>co", "<CMD>AerialToggle<CR>", desc = "Toggle Aerial" },
})
```

### blink

```lua
--| file: lua/plugins/blink.lua
require("blink.cmp").setup({
    fuzzy = { -- 下载预编译的Fuzzy以节省空间
        prebuilt_binaries = {
            force_version = "v*",
        },
    },
    cmdline = {
        -- 默认的cmdline回车按下执行命令
        -- keymap = { ["<CR>"] = { "select_and_accept", "fallback" } },
        completion = {
            list = { selection = { preselect = false, auto_insert = true } },
            menu = {
                auto_show = function()
                    return vim.fn.getcmdtype() == ":"
                end,
            },
            ghost_text = { enabled = false },
        },
    },
    keymap = {
        preset = "none",
        ["<C-space>"] = { "show", "show_documentation", "hide_documentation" },
        ["<CR>"] = { "accept", "fallback" },
        ["<C-p>"] = { "select_prev", "snippet_backward", "fallback" },
        ["<C-n>"] = { "select_next", "snippet_forward", "fallback" },
        ["<Tab>"] = {
            function(cmp)
                -- 1. 優先處理 Copilot Suggestion (Ghost Text)
                local ok, copilot = pcall(require, "copilot.suggestion")
                if ok and copilot.is_visible() then
                    -- 建立 Undo 斷點 (讓一次 undo 就能撤銷整個 AI 建議)
                    vim.api.nvim_feedkeys(vim.api.nvim_replace_termcodes("<C-G>u", true, 0, true), "n", true)
                    copilot.accept()
                    return true -- 攔截按鍵，不執行後續動作
                end
                -- 2. 如果選單開著，就選下一個
                if cmp.is_visible() then
                    return cmp.select_next()
                end
            end,
            "fallback", -- 3. 都不是的話，就執行原生的 Tab (縮排)
        },
        ["<C-b>"] = { "scroll_documentation_up", "fallback" },
        ["<C-f>"] = { "scroll_documentation_down", "fallback" },
        ["<C-e>"] = { "snippet_forward", "select_next", "fallback" },
        ["<C-u>"] = { "snippet_backward", "select_prev", "fallback" },
    },
    completion = {
        keyword = { range = "full" },
        documentation = { auto_show = true, auto_show_delay_ms = 0 },
        list = { selection = { preselect = false, auto_insert = false } },
    },
    enabled = function()
        return not vim.tbl_contains({}, vim.bo.filetype) and vim.bo.buftype ~= "prompt" and vim.b.completion ~= false
    end,
    appearance = {
        use_nvim_cmp_as_default = true,
        nerd_font_variant = "mono",
    },
    sources = {
        default = { "copilot", "lsp", "path", "snippets", "buffer" },
        providers = {
            copilot = {
                name = "copilot",
                module = "blink-copilot",
                score_offset = 100,
                async = true,
            },
            buffer = { score_offset = 5 },
            path = { score_offset = 3 },
            lsp = { score_offset = 2 },
            snippets = { score_offset = 1 },
            -- cmdline = { -- 输入超过3个及以上字母才触发补全
            --     min_keyword_length = function(ctx)
            --         if ctx.mode == "cmdline" and string.find(ctx.line, " ") == nil then return 3 end
            --         return 0
            --     end,
            -- },
        },
    },
})
```

### compile-mode

```lua
--file: lua/plugins/compile-mode.lua
vim.g.baleia = require("baleia").setup({})
-- Command to colorize the current buffer
vim.api.nvim_create_user_command("BaleiaColorize", function()
    vim.g.baleia.once(vim.api.nvim_get_current_buf())
end, { bang = true })
-- Command to show logs
vim.api.nvim_create_user_command("BaleiaLogs", vim.cmd.messages, { bang = true })

require("which-key").add({
    {
        "<leader>cc",
        "<CMD>Compile<CR>",
        desc = "Compile command",
    },
})
vim.g.compile_mode = {
    input_word_completion = true,
    baleia_setup = true,
    bang_expansion = true,
    default_command = {
        python = "uv run main.py",
        rust = "cargo run",
        c = "make",
        typst = "typst compile main.typ",
    },
}
```

### confrom

```lua
--| file: lua/plugins/confrom.lua
local formatters_by_ft = {
    c = { "clang-format", lsp_format = "fallback" },
    cpp = { "clang-format", lsp_format = "fallback" },
    haskell = { "fourmolu", lsp_format = "fallback" },
    lua = { "stylua", lsp_format = "fallback" },
    python = { "ruff_organize_imports", "ruff_format" },
    rust = { "rustfmt", lsp_format = "fallback" },
    tcl = { "tclint", lsp_format = "fallback" },
    tex = { "tex-fmt", lsp_format = "fallback" },
    toml = { "taplo", lsp_format = "fallback" },
    typst = { "typstyle", lsp_format = "fallback" },
    markdown = { "markdown-oxide", lsp_format = "fallback" },
}

require("conform").setup({
    formatters_by_ft = formatters_by_ft,
    format_on_save = {
        timeout_ms = 500,
        lsp_format = "fallback",
    },
})
```

### Copilot

```lua
--| file: lua/plugins/copilot.lua
require("copilot").setup({
    node_command = "/opt/homebrew/bin/node",
    filetypes = {
        markdown = true,
    },
})

vim.api.nvim_create_autocmd("User", {
    pattern = "BlinkCmpMenuOpen",
    callback = function()
        vim.b.copilot_suggestion_hidden = true
    end,
})

vim.api.nvim_create_autocmd("User", {
    pattern = "BlinkCmpMenuClose",
    callback = function()
        vim.b.copilot_suggestion_hidden = false
    end,
})
```

### Edgy

```lua
--| file: lua/plugins/edgy.lua
require("edgy").setup({
    left = {
        {
            ft = "aerial",
            size = { width = 0.2 },
        },
    },
    bottom = {
        {
            ft = "compilation",
            size = { height = 0.2 },
        },
    },
})
```

### flash

```lua
--| file: lua/plugins/flash.lua
require("flash").setup()
local wk = require("which-key")
wk.add({
    {
        "gw", 
        function() 
            require("flash").jump() 
        end, 
        desc = "Flash", 
        mode = { "n", "x", "o" },
    },
    {
        "gW",
        function()
        require("flash").treesitter()
        end,
        desc = "Flash Treesitter",
        mode = { "n", "x", "o" },
    },
})
```

### Indent-blankline

```lua
--file: lua/plugins/indent-blankline.lua
require("ibl").setup({
    indent = { char = "╎" },
})
```

### Tiny Inline Diagnostic

```lua
--\ file: lua/plugins/tiny-inline-diagnostic.lua
require("tiny-inline-diagnostic").setup({
    preset = "classic",
    options = {
        multilines = {
            enabled = true,
            always_show = true,
        },
        show_all_diags_on_cursorline = true,
        throttle = 100,
    },
})
vim.diagnostic.config({ virtual_text = false })
```

### lualine

```lua
--| file: lua/plugins/lualine.lua
require("lualine").setup({
    options = {
        -- component_separators = "|",
        component_separators = "|",
        section_separators = "",
        globalstatus = true,
    },
    sections = {
        lualine_a = {
            {
                "mode",
                fmt = function(str)
                    return str:sub(1, 3)
                end,
                right_padding = 2,
            },
        },
        lualine_b = {
            {
                "diagnostics",
                sources = { "nvim_lsp", "nvim_diagnostic" },
            },
            {
                "diff",
                symbols = {
                    added = " ",
                    modified = " ",
                    removed = " ",
                },
            },
            { "branch", icon = "" },
        },
        lualine_c = {
            {
                "filename",
                file_status = true, -- Displays file status (readonly status, modified status)
                newfile_status = true, -- Display new file status (new file means no write after created)
                path = 1, -- 0: Just the filename
                -- 1: Relative path
                -- 2: Absolute path
                -- 3: Absolute path, with tilde as the home directory
                -- 4: Filename and parent dir, with tilde as the home directory

                shorting_target = 40, -- Shortens path to leave 40 spaces in the window
                -- for other components. (terrible name, any suggestions?)
                -- It can also be a function that returns
                -- the value of `shorting_target` dynamically.
                symbols = {
                    modified = "[]", -- Text to show when the file is modified.
                    readonly = "[]", -- Text to show when the file is non-modifiable or readonly.
                    unnamed = "[]", -- Text to show for unnamed buffers.
                    newfile = "[󰎔]", -- Text to show for newly created file before first write
                },
            },
            { "encoding", padding = { left = 1, right = 1 } },
        },
        lualine_x = {
            {
                function()
                    local res = require("noice").api.status.mode.get()
                    if res and string.find(res, "recording", 1, true) then
                        return res
                    else
                        return ""
                    end
                end,
                cond = require("noice").api.status.mode.has,
                color = "Keyword",
            },
            -- {
            --     function()
            --         local keys = require("noice").api.status.command.get()
            --         return "   " .. keys
            --     end,
            --     cond = require("noice").api.status.command.has,
            --     color = "Keyword",
            -- },
        },
        lualine_y = {
            { "lsp_status", icon = "" },
        },
        lualine_z = {
            { "location" },
            { "progress" },
        },
    },
    extensions = {
        "aerial",
        "oil",
        "mason",
    },
})
```

### Mason

```lua
--| file: lua/plugins/mason.lua
require("mason").setup()
require("mason-lspconfig").setup({
    ensure_install = { "lua_ls", "rust_analyzer", "ruff" },
})
vim.lsp.config["pyright"] = {
    settings = {
        pyright = {
            disableDignostics = true,
        },
        python = {
            analysis = {
                typeCheckingMode = "off",
                reportPrivateImportUsage = "none",
                reportUnusedVariable = "none",
            },
        },
    },
}
```

### Neoscroll

```lua
--| file: lua/plugins/neoscroll.lua
require("neoscroll").setup()
```

### Noice

```lua
--| file: lua/plugins/noice.lua
require("noice").setup({
    cmdline = {
        view = "cmdline",
        format = {
            cmdline = { pattern = "^:", icon = "", lang = "vim" },
            search_down = { kind = "search", pattern = "^/", icon = " ", lang = "regex" },
            search_up = { kind = "search", pattern = "^%?", icon = " ", lang = "regex" },
            filter = { pattern = "^:%s*!", icon = "$", lang = "bash" },
            lua = {
                pattern = { "^:%s*lua%s+", "^:%s*lua%s*=%s*", "^:%s*=%s*" },
                icon = "",
                lang = "lua",
            },
            help = { pattern = "^:%s*he?l?p?%s+", icon = "" },
            input = { view = "cmdline", icon = "󰥻 " }, -- Used by input()
        },
    },
    lsp = {
        -- override markdown rendering so that **cmp** and other plugins use **Treesitter**
        override = {
            ["vim.lsp.util.convert_input_to_markdown_lines"] = true,
            ["vim.lsp.util.stylize_markdown"] = true,
            ["cmp.entry.get_documentation"] = true, -- requires hrsh7th/nvim-cmp
        },
    },
    popumenu = {
        enabled = false,
    },
    -- you can enable a preset for easier configuration
    presets = {
        bottom_search = true, -- use a classic bottom cmdline for search
        command_palette = true, -- position the cmdline and popupmenu together
        long_message_to_split = true, -- long messages will be sent to a split
        inc_rename = false, -- enables an input dialog for inc-rename.nvim
        lsp_doc_border = false, -- add a border to hover docs and signature help
    },
    routes = {
        {
            filter = { event = "msg_show", kind = { "shell_out", "shell_err" } },
            view = "split",
            opts = {
                level = "info",
                skip = false,
                replace = false,
            },
        },
    },
})
```

### Nvim autopairs

```lua
--| file: lua/plugins/nvim-autopairs.lua
local Rule = require("nvim-autopairs.rule")
local npairs = require("nvim-autopairs")
npairs.setup({
    disable_filetype = { "Oil" },
})
npairs.add_rules({
    Rule("$$", "$$", "tex"),
    Rule("\\begin", "\\end", "tex"),
    Rule("function", "end", "lua"),
})
```

### Nvim Surround

```lua
--| file: lua/plugins/nvim-surround.lua
require("nvim-surround").setup({})
```

### Oil

```lua
--| file: lua/plugins/oil.lua
require("oil").setup({
    columns = {
        "permissions",
        "size",
        "mtime",
        "icon",
    },
})
local wk = require("which-key")
wk.add({
    {
        "<leader>.",
        "<CMD>Oil<CR>",
        desc = "Open Oil",
        icon = "",
        noremap = true,
        silent = true,
    },
})
```

### Origami

```lua
--| file: lua/plugins/origami.lua
vim.opt.foldlevel = 99
vim.opt.foldlevelstart = 99
require("origami").setup({
    useLspFoldsWithTreesitterFallback = {
        enabled = true,
        foldmethodIfNeitherIsAvailable = "indent", ---@type string|fun(bufnr: number): string
    },
    pauseFoldsOnSearch = true,
    foldtext = {
        enabled = true,
        padding = {
            character = " ",
            width = 3, ---@type number|fun(win: number, foldstart: number, currentVirtualTextLength: number): number
            hlgroup = nil,
        },
        lineCount = {
            template = "%d lines", -- `%d` is replaced with the number of folded lines
            hlgroup = "Comment",
        },
        diagnosticsCount = true, -- uses hlgroups and icons from `vim.diagnostic.config().signs`
        gitsignsCount = true, -- requires `gitsigns.nvim`
        disableOnFt = { "snacks_picker_input" }, ---@type string[]
    },
    autoFold = {
        enabled = false,
        kinds = { "comment", "imports" }, ---@type lsp.FoldingRangeKind[]
    },
    foldKeymaps = {
        setup = true, -- modifies `h`, `l`, `^`, and `$`
        closeOnlyOnFirstColumn = true, -- `h` and `^` only fold in the 1st column
        scrollLeftOnCaret = false, -- `^` should scroll left (basically mapped to `0^`)
    },
})
```

### Presence

```lua
--| file: lua/plugins/presence.lua
require("presence").setup({
    -- General options
    auto_update = true, -- Update activity based on autocmd events (if `false`, map or manually execute `:lua package.loaded.presence:update()`)
    neovim_image_text = "The One True Text Editor", -- Text displayed when hovered over the Neovim image
    main_image = "neovim", -- Main image display (either "neovim" or "file")
    -- client_id = "793271441293967371", -- Use your own Discord application client id (not recommended)
    log_level = nil, -- Log messages at or above this level (one of the following: "debug", "info", "warn", "error")
    debounce_timeout = 10, -- Number of seconds to debounce events (or calls to `:lua package.loaded.presence:update(<filename>, true)`)
    enable_line_number = false, -- Displays the current line number instead of the current project
    blacklist = {}, -- A list of strings or Lua patterns that disable Rich Presence if the current file name, path, or workspace matches
    buttons = true, -- Configure Rich Presence button(s), either a boolean to enable/disable, a static table (`{{ label = "<label>", url = "<url>" }, ...}`, or a function(buffer: string, repo_url: string|nil): table)
    file_assets = {}, -- Custom file asset definitions keyed by file names and extensions (see default config at `lua/presence/file_assets.lua` for reference)
    show_time = true, -- Show the timer

    -- Rich Presence text options
    editing_text = "Editing %s", -- Format string rendered when an editable file is loaded in the buffer (either string or function(filename: string): string)
    file_explorer_text = "Browsing %s", -- Format string rendered when browsing a file explorer (either string or function(file_explorer_name: string): string)
    git_commit_text = "Committing changes", -- Format string rendered when committing changes in git (either string or function(filename: string): string)
    plugin_manager_text = "Managing plugins", -- Format string rendered when managing plugins (either string or function(plugin_manager_name: string): string)
    reading_text = "Reading %s", -- Format string rendered when a read-only or unmodifiable file is loaded in the buffer (either string or function(filename: string): string)
    workspace_text = "Working on %s", -- Format string rendered when in a git repository (either string or function(project_name: string|nil, filename: string): string)
    line_number_text = "Line %s out of %s", -- Format string rendered when `enable_line_number` is set to true (either string or function(line_number: number, line_count: number): string)
})
```

### Telescope

```lua
--| file: lua/plugins/telescope.lua
local telescope = require("telescope.builtin")
local wk = require("which-key")
wk.add({
    {
        "<leader>f",
        group = "Telescope",
        icon = "",
    },
    {
        "<leader>ff",
        telescope.find_files,
        desc = "Telescope find files",
        icon = "",
        noremap = true,
        silent = true,
    },
    {
        "<leader>fg",
        telescope.live_grep,
        desc = "Telescope live grep",
        icon = "",
        noremap = true,
        silent = true,
    },
    {
        "<leader>fb",
        telescope.buffers,
        desc = "Telescope buffers",
        icon = "",
        noremap = true,
        silent = true,
    },
    {
        "<leader>b",
        telescope.buffers,
        desc = "Telescope buffers",
        icon = "",
        noremap = true,
        silent = true,
    },
    {
        "<leader>fh",
        telescope.help_tags,
        desc = "Telescope help tags",
        icon = "",
        noremap = true,
        silent = true,
    },
})
```

### Tv

```lua
--| file: lua/plugins/tv.lua
require("tv").setup()

local wk = require("which-key")
wk.add({
    {
        "<leader>tv",
        desc = "TV: Select channel",
        icon = "",
        noremap = true,
        silent = true,
    },
})
```

### Typst Preview


```lua
--| file: lua/plugins/typst-preview.lua
require("typst-preview").setup()
require("which-key").add({
    {
        "<leader>cp",
        "<CMD>TypstPreview<CR>",
        desc = "preview typst file in browser",
        icon = "",
    },
})
```

### which-key

```lua
--| file: lua/plugins/which-key.lua
local function create_scratch_buffer()
    local buf = vim.api.nvim_create_buf(false, true)
    vim.api.nvim_buf_set_name(buf, "[Scratch]")
    vim.api.nvim_buf_set_option(buf, "buftype", "nofile")
    vim.api.nvim_buf_set_option(buf, "bufhidden", "hide")
    vim.api.nvim_buf_set_option(buf, "swapfile", false)
    vim.api.nvim_buf_set_option(buf, "filetype", "markdown")
    vim.api.nvim_set_current_buf(buf)
end

local wk = require("which-key")
wk.add({
    {
        "<leader>s",
        create_scratch_buffer,
        desc = "Open scratch buffer",
        noremap = true,
        silent = true,
    },
    {
        "yL",
        "<CMD>t.<CR>",
        desc = "yank line and paste below",
        noremap = true,
        silent = true,
    },
    {
        "<leader>g",
        group = "Go to",
    },
    {
        "<leader>c",
        group = "Code",
        icon = "",
    },
    {
        "<leader>ca",
        vim.lsp.buf.code_action,
        desc = "Code Action",
        icon = "",
        noremap = true,
        silent = true,
    },
    {
        "<leader>w",
        group = "Window",
        icon = "󱂬",
    },
    {
        "<leader>ws",
        "<C-w>s",
        desc = "New window Horizontal",
    },
    {
        "<leader>wv",
        "<C-w>v",
        desc = "New window Vertical",
    },
    {
        "<leader>wq",
        "<C-w>q",
        desc = "Close current window",
    },
    {
        "<leader>wJ",
        "<C-w>J",
        desc = "Move window to down",
    },
    {
        "<leader>wK",
        "<C-w>K",
        desc = "Move window to up",
    },
    {
        "<leader>wH",
        "<C-w>H",
        desc = "Move window to left",
    },
    {
        "<leader>wL",
        "<C-w>L",
        desc = "Move window to right",
    },
    {
        "<leader>wh",
        "<C-w>h",
        desc = "Go to left window",
    },
    {
        "<leader>wj",
        "<C-w>j",
        desc = "Go to down window",
    },
    {
        "<leader>wk",
        "<C-w>k",
        desc = "Go to up window",
    },
    {
        "<leader>wl",
        "<C-w>l",
        desc = "Go to right window",
    },
    {
        "<leader>w-",
        "<C-w>-",
        desc = "Decrease window height",
    },
    {
        "<leader>w+",
        "<C-w>+",
        desc = "Increase window height",
    },
    {
        "<leader>w>",
        "<C-w>>",
        desc = "Increase window height",
    },
    {
        "<leader>w<",
        "<C-w><",
        desc = "Decrease window height",
    },
    {
        "<C-l>",
        "<C-w>l",
    },
    {
        "<C-h>",
        "<C-w>h",
    },
    {
        "<C-j>",
        "<C-w>j",
    },
    {
        "<C-k>",
        "<C-w>k",
    },
    {
        "<leader>k",
        vim.lsp.buf.hover,
        desc = "LSP Hover",
        icon = "",
        noremap = true,
        silent = true,
    },
    {
        "gd",
        vim.lsp.buf.definition,
        desc = "Go to definition",
        icon = "",
        noremap = true,
        silent = true,
    },
    {
        "gD",
        vim.lsp.buf.declaration,
        desc = "Go to declaration",
        icon = "",
        noremap = true,
        silent = true,
    },
    {
        "go",
        vim.lsp.buf.type_definition,
        desc = "Go to type definition",
        icon = "",
        noremap = true,
        silent = true,
    },
    {
        "<leader>cr",
        vim.lsp.buf.rename,
        desc = "LSP Rename",
        icon = "",
        noremap = true,
        silent = true,
    },
    {
        "<leader>d",
        vim.diagnostic.open_float,
        desc = "Open diagnostics",
        icon = "",
        noremap = true,
        silent = true,
    },
    {
        "<leader>c-",
        vim.diagnostic.goto_prev,
        noremap = true,
        silent = true,
    },
    {
        "<leader>c=",
        vim.diagnostic.goto_next,
        noremap = true,
        silent = true,
    },
    {
        "<leader>a",
        "<CMD>Copilot disable<CR>",
        desc = "Disable Copilot",
        icon = "",
        noremap = true,
        silent = true,
    },
    {
        "<leader>h",
        "<CMD>nohlsearch<CR>",
        desc = "Clear search highlight",
        icon = "󰸱",
        noremap = true,
        silent = true,
    },
    {
        "H",
        "<CMD>bp<CR>",
        noremap = true,
        silent = true,
    },
    {
        "L",
        "<CMD>bn<CR>",
        noremap = true,
        silent = true,
    },
})

```
