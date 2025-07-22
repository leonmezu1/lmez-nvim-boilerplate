# 🎨 Configuration Patterns

Understanding common patterns used in Neovim configurations will help you read, modify, and extend your setup effectively. This guide covers the most important patterns you'll encounter.

## 🔧 Plugin Configuration Patterns

### 1. **Basic Plugin Setup**
```lua
{
  "author/plugin-name",
  opts = {
    option1 = true,
    option2 = "value"
  }
}
```
**When to use:** Simple plugins with straightforward configuration options.

### 2. **Custom Configuration Function**
```lua
{
  "author/plugin-name",
  config = function(_, opts)
    local plugin = require("plugin-name")
    plugin.setup(opts)
    
    -- Additional custom setup
    vim.keymap.set("n", "<leader>x", plugin.some_function)
  end
}
```
**When to use:** When you need custom logic beyond simple options.

### 3. **Dependency Management**
```lua
{
  "main-plugin",
  dependencies = {
    "required-plugin-1",
    "required-plugin-2",
    {
      "optional-plugin",
      config = function()
        -- Configure the dependency
      end
    }
  }
}
```
**Pattern:** Dependencies load before the main plugin automatically.

## ⚡ Lazy Loading Patterns

### 1. **Event-Based Loading**
```lua
{
  "plugin-name",
  event = "BufReadPost"        -- Load when opening existing files
}

-- Common events:
-- "BufReadPost"     - After reading a file
-- "BufNewFile"      - When creating a new file  
-- "InsertEnter"     - When entering insert mode
-- "VeryLazy"        - After startup is complete
```

### 2. **Key-Based Loading**
```lua
{
  "plugin-name", 
  keys = {
    { "<leader>ff", "<cmd>SomeCommand<cr>", desc = "Find Files" },
    { "<leader>fg", function() require("plugin").method() end, desc = "Grep" }
  }
}
```
**Pattern:** Plugin loads only when these keys are pressed.

### 3. **Command-Based Loading**
```lua
{
  "plugin-name",
  cmd = { "Command1", "Command2" }
}
```
**Pattern:** Plugin loads when these commands are run.

### 4. **Filetype-Based Loading**
```lua
{
  "plugin-name",
  ft = { "javascript", "typescript", "jsx" }
}
```
**Pattern:** Plugin loads only for specific file types.

## 🎯 Keymap Patterns

### 1. **Leader Key Organization**
```lua
-- File operations
"<leader>f" + something    -- <leader>ff, <leader>fg, <leader>fr
-- Buffer operations  
"<leader>b" + something    -- <leader>bd, <leader>bp, <leader>bn
-- Code operations
"<leader>c" + something    -- <leader>ca, <leader>cr, <leader>cf
-- Git operations
"<leader>g" + something    -- <leader>gs, <leader>gc, <leader>gd
```

### 2. **Mode-Specific Keymaps**
```lua
keys = {
  { "j", "gj", mode = "n", desc = "Better down movement" },
  { "k", "gk", mode = "n", desc = "Better up movement" },
  { "<leader>ca", vim.lsp.buf.code_action, mode = { "n", "v" } }
}
```

### 3. **Conditional Keymaps**
```lua
-- LSP keymaps (only when LSP is active)
vim.api.nvim_create_autocmd("LspAttach", {
  callback = function(args)
    local client = vim.lsp.get_client_by_id(args.data.client_id)
    local buffer = args.buf
    
    if client.server_capabilities.hoverProvider then
      vim.keymap.set("n", "K", vim.lsp.buf.hover, { buffer = buffer })
    end
  end
})
```

## 🔄 Autocommand Patterns

### 1. **File Type Specific Setup**
```lua
vim.api.nvim_create_autocmd("FileType", {
  pattern = { "javascript", "typescript" },
  callback = function()
    vim.opt_local.shiftwidth = 2
    vim.opt_local.tabstop = 2
  end
})
```

### 2. **Event-Based Actions**
```lua
-- Format on save
vim.api.nvim_create_autocmd("BufWritePre", {
  callback = function()
    vim.lsp.buf.format({ async = false })
  end
})

-- Highlight on yank
vim.api.nvim_create_autocmd("TextYankPost", {
  callback = function()
    vim.highlight.on_yank()
  end
})
```

### 3. **Grouped Autocommands**
```lua
local function augroup(name)
  return vim.api.nvim_create_augroup("config_" .. name, { clear = true })
end

vim.api.nvim_create_autocmd("BufReadPost", {
  group = augroup("last_loc"),
  callback = function()
    -- Go to last cursor position
  end
})
```

## 🎛️ Option Setting Patterns

### 1. **Basic Options**
```lua
vim.opt.number = true           -- Enable line numbers
vim.opt.relativenumber = true   -- Enable relative line numbers
vim.opt.mouse = "a"             -- Enable mouse support
```

### 2. **List Options**
```lua
vim.opt.shortmess:append("c")   -- Add to existing list
vim.opt.runtimepath:prepend(path) -- Add to beginning of list
```

### 3. **Buffer/Window Local Options**
```lua
vim.opt_local.wrap = true       -- Only for current buffer
vim.wo.wrap = true              -- Only for current window
vim.bo.expandtab = true         -- Only for current buffer
```

## 🔗 Function Definition Patterns

### 1. **Utility Functions**
```lua
local M = {}

function M.has_plugin(plugin)
  return require("lazy.core.config").spec.plugins[plugin] ~= nil
end

function M.get_root()
  return vim.fn.getcwd()
end

return M
```

### 2. **Configuration Functions**
```lua
local function setup_lsp_keymaps(client, buffer)
  local function map(mode, key, action, desc)
    vim.keymap.set(mode, key, action, { buffer = buffer, desc = desc })
  end
  
  map("n", "gd", vim.lsp.buf.definition, "Go to Definition")
  map("n", "K", vim.lsp.buf.hover, "Hover Documentation")
end
```

### 3. **Plugin Setup Functions**
```lua
return {
  "plugin-name",
  opts = function()
    -- Dynamic configuration
    return {
      theme = vim.o.background == "dark" and "dark" or "light",
      width = vim.o.columns > 120 and 120 or vim.o.columns
    }
  end
}
```

## 🛡️ Error Handling Patterns

### 1. **Safe Require**
```lua
local ok, plugin = pcall(require, "plugin-name")
if not ok then
  vim.notify("Plugin not found: plugin-name", vim.log.levels.WARN)
  return
end
```

### 2. **Safe Function Calls**
```lua
-- In our LSP config fix:
for _, server in ipairs(ensure_installed) do
  local ok, _ = pcall(setup, server)
  if not ok then
    vim.notify("Failed to setup " .. server, vim.log.levels.WARN)
  end
end
```

### 3. **Graceful Degradation**
```lua
local function format_buffer()
  if vim.lsp.get_active_clients()[1] then
    vim.lsp.buf.format()
  else
    -- Fallback to basic formatting
    vim.cmd("normal! gg=G")
  end
end
```

## 🧩 Common Plugin Integration Patterns

### 1. **LSP Integration**
```lua
-- Many plugins integrate with LSP
{
  "telescope.nvim",
  keys = {
    { "gd", "<cmd>Telescope lsp_definitions<cr>" },
    { "gr", "<cmd>Telescope lsp_references<cr>" }
  }
}
```

### 2. **Which-Key Integration**
```lua
{
  "which-key.nvim",
  opts = {
    spec = {
      { "<leader>f", group = "file/find" },
      { "<leader>g", group = "git" },
      { "<leader>c", group = "code" }
    }
  }
}
```

### 3. **Statusline Integration**
```lua
{
  "lualine.nvim",
  opts = {
    sections = {
      lualine_x = {
        -- Shows noice command status
        function() 
          return require("noice").api.status.command.get() 
        end,
        cond = function() 
          return package.loaded["noice"] 
        end
      }
    }
  }
}
```

## 📊 Performance Patterns

### 1. **Defer Heavy Operations**
```lua
vim.defer_fn(function()
  -- Heavy initialization here
  require("heavy-plugin").setup()
end, 100)
```

### 2. **Conditional Loading**
```lua
{
  "plugin-name",
  cond = function()
    return vim.fn.executable("some-binary") == 1
  end
}
```

### 3. **Disable Unused Built-ins**
```lua
require("lazy").setup("plugins", {
  performance = {
    rtp = {
      disabled_plugins = {
        "netrwPlugin",  -- We use neo-tree
        "tutor",        -- Don't need built-in tutor
        "zipPlugin"     -- Rarely needed
      }
    }
  }
})
```

## 🎨 Customization Patterns

### 1. **Theme Switching**
```lua
-- In colorscheme.lua
local function get_theme()
  local hour = tonumber(os.date("%H"))
  return hour >= 6 and hour < 18 and "light" or "dark"
end

return {
  "theme-plugin",
  opts = {
    style = get_theme()
  }
}
```

### 2. **Environment-Specific Config**
```lua
local is_work = vim.fn.hostname():match("work%-")
local config = {
  -- Base config
}

if is_work then
  config.corporate_proxy = true
  config.restricted_plugins = false
end
```

### 3. **User Override Pattern**
```lua
-- Allow local overrides
local ok, local_config = pcall(require, "config.local")
if ok then
  config = vim.tbl_deep_extend("force", config, local_config)
end
```

## 🧪 Testing Patterns

### 1. **Feature Detection**
```lua
local has_nvim_010 = vim.fn.has("nvim-0.10") == 1
if has_nvim_010 then
  -- Use new API
else
  -- Use compatibility layer
end
```

### 2. **Plugin Availability**
```lua
local function setup_if_available(plugin_name, setup_fn)
  local ok = pcall(require, plugin_name)
  if ok then
    setup_fn()
  end
end
```

## 💡 Pro Tips

### 1. **Use Functions for Complex Logic**
```lua
-- Instead of complex inline configuration
config = function(_, opts)
  -- Complex setup logic here
  return processed_opts
end
```

### 2. **Group Related Configuration**
```lua
-- Keep related settings together
local editor_opts = {
  number = true,
  relativenumber = true,
  signcolumn = "yes"
}

for key, value in pairs(editor_opts) do
  vim.opt[key] = value
end
```

### 3. **Document Your Patterns**
```lua
-- Always explain complex patterns
-- This pattern allows lazy loading while maintaining global access
_G.my_plugin = setmetatable({}, {
  __index = function(_, key)
    return require("actual-plugin")[key]
  end
})
```

**Next:** [Customization Guide](04-customization.md) - Learn how to make this configuration your own.