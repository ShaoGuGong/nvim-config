# ColorScheme

---

## Evergarden

```lua
--| file: lua/plugins/colorschemes.lua
require("evergarden").setup({
	theme = {
		variant = "summer", -- 'winter'|'fall'|'spring'|'summer'
		accent = "green",
	},
	editor = {
		transparent_background = false,
		sign = { color = "none" },
		float = {
			color = "mantle",
			solid_border = false,
		},
		completion = {
			color = "surface0",
		},
	},
})
```

## Nordic

```lua
--| file: lua/plugins/colorschemes.lua
require("nordic").setup({
	bold_keywords = true,
	transparent = {
		bg = false,
		float = false,
	},
})
```

## Kanagawa

```lua
--| file: lua/plugins/colorschemes.lua
require("kanagawa").setup({
	compile = false, -- enable compiling the colorscheme
	undercurl = true, -- enable undercurls
	commentStyle = { italic = true },
	functionStyle = {},
	keywordStyle = { italic = true },
	statementStyle = { bold = true },
	typeStyle = {},
	transparent = true, -- do not set background color
	dimInactive = false, -- dim inactive window `:h hl-NormalNC`
	terminalColors = true, -- define vim.g.terminal_color_{0,17}
	colors = { -- add/modify theme and palette colors
		palette = {},
		theme = { wave = {}, lotus = {}, dragon = {}, all = {
			ui = {
				bg_gutter = "none",
			},
		} },
	},
	overrides = function(colors) -- add/modify highlights
		return {}
	end,
	theme = "wave", -- Load "wave" theme
	background = { -- map the value of 'background' option to a theme
		dark = "wave", -- try "dragon" !
		light = "lotus",
	},
})
```

## VSCode

```lua
--| file: lua/plugins/colorschemes.lua
require("vscode").setup({
	transparent = false,
})
```

## Gruvbox

```lua
--| file: lua/plugins/colorschemes.lua
require("gruvbox").setup({
	transparent_mode = true,
})
```

--- 

## Setting

```lua
--| file: lua/plugins/colorschemes.lua
vim.cmd("colorscheme kanagawa")
```
