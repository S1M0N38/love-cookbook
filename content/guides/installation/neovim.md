---
title: "Neovim"
date: 2026-10-06
authors: [Nykenik, S1M0N38]
---

When using Neovim for LÖVE can use S1M0N38's [`love2d.nvim`](https://github.com/S1M0N38/love2d.nvim) plugin.

> [!IMPORTANT]
> This guide covers the setup. The plugin also ships full documentation, so run `:help love2d` to read it. Learning to navigate `:help` is one of the most valuable Neovim skills you can pick up.

## Disclaimer
This guide already supposes you have [LÖVE installed](/guides/installation/love/), [lua_ls](https://luals.github.io/) installed and a plugin manager. Neovim 0.12.2 or newer is required, so update Neovim before installing if you're on an older version.

You don't need `nvim-lspconfig` anymore. love2d.nvim configures `lua_ls` with Neovim's built-in LSP functions, so there is nothing to wire up by hand.

If you don't have any of these, the easiest way is to install an already made Neovim configuration, or a "Neovim distribution".

If something doesn't work, run `:checkhealth love2d`. It checks your Neovim version, the LÖVE binary, `lua_ls`, the type definitions and the Treesitter parsers.

## Setup

### Installation
The plugin works with any plugin manager, or with none at all. We are going to use `lazy.nvim` here.

> [!IMPORTANT]
> love2d.nvim 3.0.0 renamed its commands. `:LoveRun` and `:LoveStop` are gone; use `:Love run` and `:Love stop` instead. If you followed an older version of this guide, update your keymaps.

If you have a single-file setup, go to your `init.lua` and add this to your lazy setup:
```lua
require("lazy").setup({
  -- Other plugins go here.
  {
    "S1M0N38/love2d.nvim",
    version = "3.*",
    event = "VeryLazy",
    opts = {},
    keys = {
      { "<leader>v", "", desc = "LÖVE", ft = "lua" },
      { "<leader>vr", "<cmd>Love run<cr>", desc = "Run LÖVE", ft = "lua" },
      { "<leader>vw", "<cmd>Love watch<cr>", desc = "Watch LÖVE", ft = "lua" },
      { "<leader>vi", "<cmd>Love info<cr>", desc = "Info LÖVE", ft = "lua" },
      { "<leader>vs", "<cmd>Love stop<cr>", desc = "Stop LÖVE", ft = "lua" },
      { "<leader>vo", "<cmd>Love output<cr>", desc = "Output panel", ft = "lua" },
    },
  },
})
```

Otherwise, if you have a multi-file setup, create a new file in your plugins folder called `love2d.lua` and copy this into it:
```lua
return {
  "S1M0N38/love2d.nvim",
  version = "3.*",
  event = "VeryLazy",
  opts = {},
  keys = {
    { "<leader>v", "", desc = "LÖVE", ft = "lua" },
    { "<leader>vr", "<cmd>Love run<cr>", desc = "Run LÖVE", ft = "lua" },
    { "<leader>vw", "<cmd>Love watch<cr>", desc = "Watch LÖVE", ft = "lua" },
    { "<leader>vi", "<cmd>Love info<cr>", desc = "Info LÖVE", ft = "lua" },
    { "<leader>vs", "<cmd>Love stop<cr>", desc = "Stop LÖVE", ft = "lua" },
    { "<leader>vo", "<cmd>Love output<cr>", desc = "Output panel", ft = "lua" },
  },
}
```

**Without a plugin manager.** Neovim 0.12 ships with `vim.pack`, so you can install the plugin without adding a plugin manager at all:
```lua
vim.pack.add({
  {
    src = "https://github.com/S1M0N38/love2d.nvim",
    version = vim.version.range("3"),
  },
})
require("love2d").setup({})
vim.keymap.set("n", "<leader>vr", "<cmd>Love run<cr>", { desc = "Run LÖVE" })
vim.keymap.set("n", "<leader>vw", "<cmd>Love watch<cr>", { desc = "Watch LÖVE" })
vim.keymap.set("n", "<leader>vi", "<cmd>Love info<cr>", { desc = "Info LÖVE" })
vim.keymap.set("n", "<leader>vs", "<cmd>Love stop<cr>", { desc = "Stop LÖVE" })
vim.keymap.set("n", "<leader>vo", "<cmd>Love output<cr>", { desc = "Output panel" })
```

### Post-installation
After installing the plugin, we can configure it. It has three options:
- `path_to_love_bin`: The path to the `love` binary. Leave it as `"love"` if LÖVE is installed system-wide. On macOS, if you installed LÖVE as an `.app`, set it to `"/Applications/love.app/Contents/MacOS/love"`.
- `lsp`: If love2d.nvim should configure `lua_ls` for LÖVE projects automatically. It's `true` by default. Set it to `false` if you prefer to configure `lua_ls` yourself.
- `output`: Controls the floating output panel. It opens automatically by default. Set it to `false` to keep it closed, or pass a table of window options to customize it.

To change these values, modify the `opts` table in the plugin installation:
```lua
return {
  -- Other options go here.
  opts = {
    -- These are the default values.
    path_to_love_bin = "love",
    lsp = true,
    output = nil,
  },
  -- Keymaps go here.
}
```

## Usage
After completing the plugin's setup, we now can use it to develop games with LÖVE and Neovim.

Start Neovim from your project folder. love2d.nvim finds the project by looking for a `main.lua` or a `conf.lua` in the folder you're in, or in any of its parent folders, so you don't need to open the exact folder that holds `main.lua`.

All commands live behind a single `:Love` command with subcommands:
- `:Love run` (`<leader>vr`): Run the project.
- `:Love watch` (`<leader>vw`): Run the project and restart it every time you save a Lua file. This is the usual way to work.
- `:Love stop` (`<leader>vs`): Stop the running project, and watch mode with it.
- `:Love info` (`<leader>vi`): Show the detected project, its entry point and whether it's running.
- `:Love output` (`<leader>vo`): Show or hide the output panel with the game's `print` and error output.

> [!NOTE]
> By default LÖVE opens a graphical error screen when the game crashes, so that output never reaches Neovim. To send errors to the output panel and to your editor's diagnostics, your game needs a custom error handler — see `:help love2d-gamesetup`.

You also don't have to configure anything for `love.` completions to work. When love2d.nvim detects a LÖVE project it configures `lua_ls` with LÖVE's type definitions and the LuaJIT runtime, so completions, hover help (`K`) and go-to-definition work like any other Lua project. When you leave the project, those settings are removed again.

## GLSL highlighting
As a bonus, we can also get shader syntax highlighting.

To do this, just install the parsers for GLSL with `:TSInstall lua glsl`. After doing so, Neovim will recognize the strings passed to `love.graphics.newShader()` as GLSL code.

