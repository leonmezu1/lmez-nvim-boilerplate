# 🏗️ Architecture & Structure

Understanding how a Neovim configuration is organized is crucial for customization and maintenance. This guide explains the architectural decisions behind this setup.

## 📁 Directory Structure

```
~/.config/nvim/
├── init.lua                 # Entry point - loads everything
├── lazy-lock.json          # Plugin version lockfile (auto-generated)
└── lua/
    ├── config/
    │   ├── options.lua      # Vim options & settings
    │   ├── keymaps.lua      # Key bindings
    │   ├── autocmds.lua     # Auto commands & events  
    │   ├── icons.lua        # Icon definitions
    │   ├── util.lua         # Utility functions
    │   └── lsp/
    │       ├── keymaps.lua  # LSP-specific keybindings
    │       └── util.lua     # LSP utility functions
    └── plugins/
        ├── colorscheme.lua  # Color themes
        ├── ui.lua          # UI enhancements
        ├── editor.lua      # Editor features
        ├── coding.lua      # Code-related plugins
        ├── lsp.lua         # Language server setup
        └── extras.lua      # Additional features
```

## 🎯 Design Principles

### 1. **Separation of Concerns**
Each file has a single responsibility:
- **Core config** (options, keymaps, autocmds) are separate from plugins
- **Plugin categories** are logically grouped
- **LSP configuration** has its own module

### 2. **Modularity**
```lua
-- init.lua loads modules in order
require("config.options")    -- Load settings first
require("config.keymaps")    -- Then keybindings
require("config.autocmds")   -- Finally autocommands
```

**Why this order?**
- Options set the foundation
- Keymaps may depend on options
- Autocmds often reference both

### 3. **Lazy Loading Architecture**
```lua
-- Plugin manager setup
require("lazy").setup("plugins", {
  performance = {
    rtp = {
      disabled_plugins = { "netrwPlugin", ... } -- Disable unused built-ins
    }
  }
})
```

**Benefits:**
- **Faster startup**: Only essential plugins load immediately
- **Memory efficient**: Unused features don't consume RAM
- **Better UX**: Neovim feels instant

## 🔄 Loading Process

### 1. **Bootstrap Phase**
```lua
-- Check if plugin manager exists
local lazypath = vim.fn.stdpath("data") .. "/lazy/lazy.nvim"
if not vim.loop.fs_stat(lazypath) then
  -- Auto-install if missing
  vim.fn.system({ "git", "clone", ... })
end
```

### 2. **Configuration Loading**
```lua
vim.g.mapleader = " "        -- Set leader BEFORE plugins load
require("config.options")    -- Core Neovim settings
require("config.keymaps")    -- Key bindings  
require("config.autocmds")   -- Event handlers
```

### 3. **Plugin Loading**
```lua
require("lazy").setup("plugins") -- Scans lua/plugins/*.lua files
```

## 📦 Plugin Organization

### Core Categories

#### **UI Plugins** (`ui.lua`)
Visual enhancements that don't affect editing behavior:
- Statusline, notifications, dashboard
- Icons, colors, visual indicators

#### **Editor Plugins** (`editor.lua`)  
Core editing functionality:
- File explorer, fuzzy finder
- Git integration, session management

#### **Coding Plugins** (`coding.lua`)
Development-specific features:
- Syntax highlighting (TreeSitter)
- Auto-completion, snippets
- Code formatting, commenting

#### **LSP Plugins** (`lsp.lua`)
Language server integration:
- LSP client configuration
- Server management (Mason)
- LSP-specific utilities

## 🎛️ Configuration Patterns

### 1. **Plugin Specification**
```lua
{
  "plugin-author/plugin-name",
  version = "v1.0",           -- Pin version for stability
  dependencies = { ... },      -- Required plugins
  event = "BufReadPost",      -- When to load
  keys = { ... },             -- Keybinding-triggered loading
  cmd = "Command",            -- Command-triggered loading  
  ft = "javascript",          -- Filetype-triggered loading
  opts = { ... },             -- Plugin options
  config = function() ... end, -- Custom setup function
}
```

### 2. **Lazy Loading Strategies**
```lua
-- Load when opening any file
event = { "BufReadPost", "BufNewFile" }

-- Load when pressing specific key
keys = { "<leader>ff" }

-- Load when running command
cmd = "Telescope"

-- Load for specific filetypes only
ft = { "javascript", "typescript" }
```

### 3. **Configuration Functions**
```lua
-- Simple configuration
opts = {
  theme = "dark",
  show_line_numbers = true
}

-- Complex configuration
config = function(_, opts)
  local plugin = require("plugin-name")
  plugin.setup(opts)
  
  -- Additional customization
  vim.keymap.set("n", "<leader>x", plugin.some_function)
end
```

## 🔧 Key Architecture Decisions

### Why Lazy.nvim?
- **Modern**: Built specifically for Neovim
- **Fast**: Sophisticated loading strategies
- **Reliable**: Lockfile ensures reproducible installs
- **User-friendly**: Great UI and debugging tools

### Why Lua over Vimscript?
- **Performance**: Lua is significantly faster
- **Power**: Full programming language capabilities
- **Integration**: Native Neovim API access
- **Modern**: Better tooling and community support

### Why This File Structure?
- **Scalable**: Easy to add new configurations
- **Maintainable**: Changes are isolated to specific files  
- **Logical**: Related functionality is grouped together
- **Standard**: Follows Lua module conventions

## 🎓 Understanding the Flow

### When you start Neovim:
1. `init.lua` runs first
2. Leader key is set
3. Core config modules load (options → keymaps → autocmds)
4. Plugin manager initializes
5. Plugins load based on their lazy loading conditions
6. LSP servers start for detected filetypes

### When you open a file:
1. Filetype detection triggers
2. Relevant plugins load (TreeSitter, LSP, etc.)
3. Language server connects
4. Syntax highlighting activates
5. Completion becomes available

### When you press a key:
1. Keymap is checked
2. If it's a lazy-loaded plugin key, plugin loads first
3. Action executes

## 🚀 Next Steps

Now that you understand the architecture, you can:
- **Navigate confidently** through the configuration
- **Make targeted changes** without breaking things
- **Add new plugins** in the right places
- **Debug issues** by understanding the loading order

**Next:** [Plugin Breakdown](02-plugins.md) - Learn what each plugin does and why it's included.