---
url: https://dev.to/miry/effortless-formatting-for-opentofu-files-with-lazyvim-4d10
canonical_url: https://dev.to/miry/effortless-formatting-for-opentofu-files-with-lazyvim-4d10
title: Effortless Formatting for OpenTofu Files with LazyVim
slug: effortless-formatting-for-opentofu-files-with-lazyvim-4d10
description: Discover a streamlined solution for automatically formatting HCL files
  in Neovim.
tags:
- opentofu
- vim
- editor
- snippet
author: Michael Nikitochkin
username: miry
---

# Effortless Formatting for OpenTofu Files with LazyVim


Here's my snippet to automatically format HCL files upon saving:

```lua
-- ~/.config/nvim/lua/plugins/hcl.lua
-- Configure automatic formatting for HCL files in NeoVim

return {
  {
    "stevearc/conform.nvim",
    opts = {
      formatters_by_ft = {
        tf = { "tfmt" },
        terraform = { "tfmt" },
        hcl = { "tfmt" },
      },
      formatters = {
        tfmt = {
          -- Specify the command and its arguments for formatting
          command = "tofu",
          args = { "fmt", "-" },
          stdin = true,
        },
      },
    },
  },
  {
    "nathom/filetype.nvim",
    config = function()
      -- Setup overrides for file extensions
      require("filetype").setup({
        overrides = {
          extensions = {
            tf = "terraform",
            tfvars = "terraform",
            tfstate = "json",
          },
        },
      })
    end,
  },
}
```


