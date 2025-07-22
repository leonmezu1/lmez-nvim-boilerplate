# 🔧 Troubleshooting Guide

This guide helps you diagnose and fix common issues you might encounter with your Neovim configuration.

## 🚨 Common Issues & Solutions

### 1. **Syntax Highlighting Not Working**

**Symptoms:**
- Code appears in plain text
- No colors for different syntax elements
- Only basic highlighting

**Diagnosis:**
```vim
:checkhealth nvim-treesitter
```

**Common Causes & Fixes:**

**Missing parsers:**
```vim
:TSInstall javascript typescript tsx css html
:TSUpdate
```

**Parser compilation issues:**
- Ensure you have a C compiler installed
- On macOS: `xcode-select --install`
- On Ubuntu: `sudo apt install build-essential`

**Broken parser configuration:**
```lua
-- Check if this exists in your coding.lua
config = function(_, opts)
  -- BAD: This removes parsers
  for _, lang in ipairs(opts.ensure_installed) do
    if vim.fn.executable(lang .. "-language-server") == 0 then
      -- This is wrong! Remove this logic
    end
  end
  
  -- GOOD: Just setup parsers
  require("nvim-treesitter.configs").setup(opts)
end
```

### 2. **LSP Not Working**

**Symptoms:**
- No autocompletion
- No error highlighting
- `gd` (go to definition) doesn't work
- No hover documentation

**Diagnosis Commands:**
```vim
:LspInfo          " Check active LSP clients
:Mason            " Check installed servers  
:checkhealth lsp  " General LSP health
```

**Common Fixes:**

**Install missing language servers:**
```vim
:Mason
" Press 'i' on the server you want to install
" For React/Rails: ts_ls, ruby_lsp, cssls, html
```

**Server not starting:**
```lua
-- Check your lsp.lua servers configuration
servers = {
  ts_ls = {},  -- Make sure this entry exists
  ruby_lsp = {},
}
```

**Mason-LSPConfig errors:**
```lua
-- In lsp.lua, ensure you have the fix:
if have_mason then
  -- Manual setup to avoid automatic_enable issues
  for _, server in ipairs(ensure_installed) do
    local ok, _ = pcall(setup, server)
    if not ok then
      vim.notify("Failed to setup " .. server, vim.log.levels.WARN)
    end
  end
  
  pcall(mlsp.setup, { 
    ensure_installed = ensure_installed,
    handlers = { setup },
    automatic_installation = false
  })
end
```

### 3. **Slow Startup**

**Diagnosis:**
```vim
:Lazy profile
```

**Common Causes:**

**Too many plugins loading at startup:**
```lua
-- BAD: Loads immediately
{
  "heavy-plugin",
  config = function() ... end
}

-- GOOD: Lazy loads
{
  "heavy-plugin", 
  event = "BufReadPost",  -- Load when opening files
  config = function() ... end
}
```

**Heavy operations in init.lua:**
```lua
-- BAD: Synchronous heavy operation
vim.fn.system("heavy-command")

-- GOOD: Defer heavy operations
vim.defer_fn(function()
  vim.fn.system("heavy-command")
end, 100)
```

### 4. **Keybindings Not Working**

**Check if key is bound:**
```vim
:map <leader>ff    " Check specific mapping
:map <leader>      " Check all leader mappings
```

**Common Issues:**

**Plugin not loaded yet:**
```lua
-- If keymap is in plugin config, plugin must load first
{
  "telescope.nvim",
  keys = {
    { "<leader>ff", "<cmd>Telescope find_files<cr>" }  -- Plugin loads when key is pressed
  }
}
```

**Key conflicts:**
```vim
:map j  " See all j mappings
" Look for conflicts
```

**Leader key timing:**
```lua
-- Make sure this is set BEFORE lazy.setup()
vim.g.mapleader = " "
```

### 5. **Plugin Not Loading**

**Check plugin installation:**
```vim
:Lazy
" Look for the plugin in the list
" Red = error, Green = loaded, Blue = not loaded yet
```

**Common Causes:**

**Wrong plugin name:**
```lua
-- BAD: Typo in plugin name
{ "nvim-telescope/telescope" }  -- Missing .nvim

-- GOOD: Correct name
{ "nvim-telescope/telescope.nvim" }
```

**Dependency issues:**
```lua
-- Make sure dependencies are correct
{
  "main-plugin",
  dependencies = {
    "required-plugin",  -- Check this exists
  }
}
```

**Loading conditions not met:**
```lua
-- This only loads for JavaScript files
{
  "js-plugin",
  ft = "javascript"  -- Won't load for TypeScript!
}

-- Fix: Add more filetypes
{
  "js-plugin", 
  ft = { "javascript", "typescript", "jsx", "tsx" }
}
```

## 🔍 Diagnostic Commands

### Essential Health Checks
```vim
:checkhealth                    " Overall health
:checkhealth nvim-treesitter   " Syntax highlighting
:checkhealth lsp               " Language servers
:checkhealth mason             " LSP server manager
:checkhealth lazy              " Plugin manager
```

### Plugin-Specific Diagnostics
```vim
:Lazy                          " Plugin manager UI
:Mason                         " LSP server manager
:LspInfo                       " Active LSP clients
:TSUpdate                      " Update parsers
:TSInstallInfo                 " Parser installation status
```

### Performance Diagnostics
```vim
:Lazy profile                  " Startup time breakdown
:luafile %                     " Test current Lua file
```

## 🚨 Emergency Recovery

### 1. **Config Won't Load (Syntax Errors)**

**Minimal recovery init.lua:**
```lua
-- Save as ~/.config/nvim/init.lua to get back to basics
vim.opt.number = true
vim.opt.mouse = "a"
vim.g.mapleader = " "

-- Only load if it exists and works
pcall(function()
  require("config.options")
end)
```

### 2. **Plugin Manager Broken**

**Reset Lazy.nvim:**
```bash
rm -rf ~/.local/share/nvim/lazy/
rm -rf ~/.local/state/nvim/lazy/
```

**Then restart Neovim and plugins will reinstall.**

### 3. **LSP Completely Broken**

**Reset Mason:**
```bash
rm -rf ~/.local/share/nvim/mason/
```

**Reset LSP state:**
```vim
:LspRestart
:Mason
" Reinstall your language servers
```

## 🐛 Debug Mode

### Enable Debug Logging

**For LSP issues:**
```lua
vim.lsp.set_log_level("debug")
-- Then check: ~/.local/state/nvim/lsp.log
```

**For plugin loading:**
```lua
-- In init.lua, before lazy.setup()
require("lazy").setup("plugins", {
  debug = true,
})
```

### Verbose Mode
```bash
nvim --startuptime startup.log
# Check startup.log for timing information
```

## 📋 Common Error Messages

### "module 'plugin-name' not found"
**Cause:** Plugin not installed or wrong name
**Fix:** Check `:Lazy` and plugin name spelling

### "attempt to call field 'setup' (a nil value)"
**Cause:** Plugin loaded but not properly configured
**Fix:** Check plugin documentation for correct setup

### "LSP client 1 quit with exit code 1"
**Cause:** Language server crashed or misconfigured
**Fix:** Check `:LspLog` and server configuration

### "TreeSitter: No parser for 'filetype'"
**Cause:** Missing TreeSitter parser
**Fix:** `:TSInstall filetype`

### "mason-lspconfig: enable field nil"
**Cause:** The automatic_enable error we fixed
**Fix:** Use the corrected LSP configuration from our fixes

## 🔧 Performance Optimization

### 1. **Identify Slow Plugins**
```vim
:Lazy profile
" Look for plugins with high load times
```

### 2. **Optimize Loading**
```lua
-- Convert immediate loading to lazy loading
{
  "slow-plugin",
  event = "VeryLazy",  -- Load after startup complete
}
```

### 3. **Disable Unnecessary Features**
```lua
-- In lazy.setup()
require("lazy").setup("plugins", {
  performance = {
    rtp = {
      disabled_plugins = {
        "gzip",
        "matchit", 
        "matchparen",
        "netrwPlugin",  -- We use neo-tree
        "tarPlugin",
        "tohtml",
        "tutor",
        "zipPlugin",
      },
    },
  },
})
```

## 🎯 Specific Framework Issues

### React/TypeScript Issues

**TSX not highlighting:**
```vim
:TSInstall tsx typescript javascript
```

**ESLint/Prettier not working:**
```lua
-- In lsp.lua servers:
servers = {
  ts_ls = {
    settings = {
      typescript = {
        format = { enable = false },  -- Let Prettier handle formatting
      },
      javascript = {
        format = { enable = false },
      },
    },
  },
}
```

### Ruby/Rails Issues

**Ruby LSP not starting:**
```bash
# Make sure ruby-lsp gem is installed
gem install ruby-lsp

# For Rails projects, add to Gemfile:
group :development do
  gem "ruby-lsp", require: false
end
```

**Slow Ruby file loading:**
```lua
-- Optimize TreeSitter for Ruby
{
  "nvim-treesitter/nvim-treesitter",
  opts = {
    highlight = {
      enable = true,
      disable = function(lang, buf)
        local max_filesize = 100 * 1024 -- 100 KB
        local ok, stats = pcall(vim.loop.fs_stat, vim.api.nvim_buf_get_name(buf))
        if ok and stats and stats.size > max_filesize then
          return true
        end
      end,
    },
  }
}
```

## 📝 Creating Bug Reports

When seeking help, include:

### 1. **System Information**
```vim
:checkhealth
" Copy the output
```

### 2. **Minimal Reproduction**
Create a minimal config that reproduces the issue:
```lua
-- minimal-repro.lua
vim.g.mapleader = " "

-- Bootstrap lazy.nvim
local lazypath = "/tmp/lazy/lazy.nvim"
if not vim.loop.fs_stat(lazypath) then
  vim.fn.system({"git", "clone", "https://github.com/folke/lazy.nvim.git", lazypath})
end
vim.opt.rtp:prepend(lazypath)

-- Only the problematic plugin
require("lazy").setup({
  {
    "problematic-plugin",
    config = function()
      -- Minimal config that shows the issue
    end
  }
})
```

### 3. **Steps to Reproduce**
1. Start nvim with the minimal config: `nvim -u minimal-repro.lua`
2. Do specific action
3. Observe error

### 4. **Expected vs Actual Behavior**
- What you expected to happen
- What actually happened
- Error messages (`:messages` to see all messages)

## 🆘 Getting Help

### Communities
- **Reddit**: [r/neovim](https://reddit.com/r/neovim)
- **Discord**: Neovim Discord server
- **GitHub**: Plugin repository issues

### Documentation
- `:help` - Built-in help system
- `:help lua-guide` - Lua in Neovim
- `:help lsp` - LSP documentation
- Plugin README files

### Pro Tips
- Always search existing issues first
- Provide minimal reproduction cases
- Include your system information
- Be specific about what you've tried

**Next:** [Building Your Own](06-build-your-own.md) - Step-by-step guide to create your config from scratch.