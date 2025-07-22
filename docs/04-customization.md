# 🎨 Customization Guide

This guide shows you how to customize every aspect of your Neovim configuration. From simple tweaks to major modifications, you'll learn how to make it truly yours.

## 🎯 Quick Customizations

### 1. **Change the Colorscheme**

**Current setup:**
```lua
-- In lua/plugins/colorscheme.lua
{
  "loctvl842/monokai-pro.nvim",
  config = function(_, opts)
    require("monokai-pro").setup(opts)
    vim.cmd.colorscheme("monokai-pro")  -- This line sets the theme
  end,
}
```

**To switch themes:**
```lua
-- Replace the config function:
config = function(_, opts)
  require("catppuccin").setup(opts)
  vim.cmd.colorscheme("catppuccin")     -- Now using Catppuccin
end,

-- Or use Tokyo Night:
config = function(_, opts)
  require("tokyonight").setup(opts)
  vim.cmd.colorscheme("tokyonight")
end,
```

### 2. **Modify Key Bindings**

**Add your own keymaps:**
```lua
-- In lua/config/keymaps.lua, add:
local map = vim.keymap.set

-- Quick save with Ctrl+S
map("n", "<C-s>", "<cmd>w<cr>", { desc = "Save file" })
map("i", "<C-s>", "<Esc><cmd>w<cr>a", { desc = "Save and continue" })

-- Better escape from insert mode
map("i", "jk", "<Esc>", { desc = "Exit insert mode" })

-- Navigate between buffers
map("n", "H", "<cmd>bprevious<cr>", { desc = "Previous buffer" })
map("n", "L", "<cmd>bnext<cr>", { desc = "Next buffer" })
```

**Modify existing plugin keymaps:**
```lua
-- In the plugin configuration, change the keys table:
{
  "nvim-telescope/telescope.nvim",
  keys = {
    -- Change from <leader>ff to <leader>pf (project find)
    { "<leader>pf", "<cmd>Telescope find_files<cr>", desc = "Find Files" },
    -- Add new shortcuts
    { "<C-p>", "<cmd>Telescope find_files<cr>", desc = "Find Files" },
  }
}
```

### 3. **Adjust Visual Settings**

**Change line numbers:**
```lua
-- In lua/config/options.lua
vim.opt.relativenumber = false  -- Turn off relative numbers
vim.opt.number = true           -- Keep absolute numbers

-- Or turn off all line numbers:
vim.opt.number = false
vim.opt.relativenumber = false
```

**Modify indentation:**
```lua
-- For 4-space indentation instead of 2:
vim.opt.shiftwidth = 4
vim.opt.tabstop = 4
vim.opt.softtabstop = 4
```

**Change cursor behavior:**
```lua
-- Make cursor a block in all modes
vim.opt.guicursor = ""

-- Or more customized:
vim.opt.guicursor = "n-v-c:block,i-ci-ve:ver25,r-cr:hor20,o:hor50"
```

## 🔧 Advanced Customizations

### 1. **Add New Plugins**

**Create a new plugin file:**
```lua
-- Create lua/plugins/my-plugins.lua
return {
  -- Add a Git blame plugin
  {
    "f-person/git-blame.nvim",
    event = "BufReadPost",
    config = function()
      require("gitblame").setup({
        enabled = false,  -- Start disabled
        message_template = " <summary> • <date> • <author>",
      })
    end,
    keys = {
      { "<leader>gb", "<cmd>GitBlameToggle<cr>", desc = "Toggle Git Blame" }
    }
  },

  -- Add a markdown preview plugin
  {
    "iamcco/markdown-preview.nvim",
    ft = "markdown",
    build = function() vim.fn["mkdp#util#install"]() end,
    keys = {
      { "<leader>mp", "<cmd>MarkdownPreviewToggle<cr>", desc = "Markdown Preview" }
    }
  }
}
```

### 2. **Customize LSP Servers**

**Add more language servers:**
```lua
-- In lua/plugins/lsp.lua, modify the servers table:
servers = {
  -- Existing servers
  lua_ls = { settings = { ... } },
  ts_ls = { settings = { ... } },
  
  -- Add new servers
  rust_analyzer = {
    settings = {
      ["rust-analyzer"] = {
        cargo = { allFeatures = true },
        checkOnSave = { command = "clippy" },
      }
    }
  },
  
  gopls = {
    settings = {
      gopls = {
        analyses = { unusedparams = true },
        staticcheck = true,
      }
    }
  },
  
  -- Python
  pyright = {
    settings = {
      python = {
        analysis = {
          typeCheckingMode = "basic"
        }
      }
    }
  }
}
```

**Customize LSP behavior:**
```lua
-- Add to the LSP config function:
config = function(_, opts)
  -- Existing setup code...
  
  -- Customize diagnostic display
  vim.diagnostic.config({
    virtual_text = false,  -- Turn off inline error text
    signs = true,
    update_in_insert = false,
    underline = true,
    severity_sort = true,
    float = {
      focusable = false,
      style = "minimal",
      border = "rounded",
      source = "always",
      header = "",
      prefix = "",
    },
  })
  
  -- Customize hover window
  vim.lsp.handlers["textDocument/hover"] = vim.lsp.with(
    vim.lsp.handlers.hover, {
      border = "rounded"
    }
  )
end
```

### 3. **Create Custom Commands**

**Add to lua/config/autocmds.lua:**
```lua
-- Create custom commands
vim.api.nvim_create_user_command("FormatDisable", function(args)
  if args.bang then
    -- For all buffers
    vim.g.disable_autoformat = true
  else
    -- For current buffer
    vim.b.disable_autoformat = true
  end
end, {
  desc = "Disable autoformat-on-save",
  bang = true,
})

vim.api.nvim_create_user_command("FormatEnable", function()
  vim.b.disable_autoformat = false
  vim.g.disable_autoformat = false
end, {
  desc = "Re-enable autoformat-on-save",
})

-- Quick note taking command
vim.api.nvim_create_user_command("Note", function(args)
  local note_file = vim.fn.expand("~/notes/" .. os.date("%Y-%m-%d") .. ".md")
  vim.cmd("edit " .. note_file)
  if args.args ~= "" then
    vim.api.nvim_buf_set_lines(0, -1, -1, false, { "", "## " .. args.args, "" })
  end
end, {
  desc = "Open daily note",
  nargs = "?",
})
```

### 4. **Customize Statusline**

**Modify Lualine sections:**
```lua
-- In lua/plugins/ui.lua, find the lualine config:
opts = function()
  return {
    sections = {
      lualine_a = { "mode" },
      lualine_b = { "branch" },
      lualine_c = {
        "filename",
        -- Add current function name
        {
          function()
            local ok, navic = pcall(require, "nvim-navic")
            if ok then
              return navic.get_location()
            end
            return ""
          end,
          cond = function()
            local ok, navic = pcall(require, "nvim-navic")
            return ok and navic.is_available()
          end
        }
      },
      lualine_x = {
        -- Show macro recording status
        {
          function()
            local reg = vim.fn.reg_recording()
            if reg == "" then return "" end
            return "recording @" .. reg
          end
        },
        "encoding",
        "fileformat", 
        "filetype"
      },
      lualine_y = { "progress" },
      lualine_z = { "location" }
    }
  }
end
```

## 🎨 Theme Customization

### 1. **Custom Highlight Groups**

**Override theme colors:**
```lua
-- In lua/plugins/colorscheme.lua:
config = function(_, opts)
  require("monokai-pro").setup(opts)
  vim.cmd.colorscheme("monokai-pro")
  
  -- Custom highlights
  vim.api.nvim_set_hl(0, "Comment", { fg = "#6272a4", italic = true })
  vim.api.nvim_set_hl(0, "CursorLine", { bg = "#44475a" })
  vim.api.nvim_set_hl(0, "Visual", { bg = "#6272a4" })
  
  -- Make the background transparent
  vim.api.nvim_set_hl(0, "Normal", { bg = "NONE" })
  vim.api.nvim_set_hl(0, "NormalFloat", { bg = "NONE" })
end
```

### 2. **Conditional Theming**

**Auto dark/light mode:**
```lua
local function set_theme_based_on_time()
  local hour = tonumber(os.date("%H"))
  if hour >= 6 and hour < 18 then
    vim.o.background = "light"
    vim.cmd.colorscheme("github_light")
  else
    vim.o.background = "dark"
    vim.cmd.colorscheme("tokyonight")
  end
end

-- Set theme on startup
set_theme_based_on_time()

-- Optional: Update every hour
vim.defer_fn(function()
  vim.fn.timer_start(3600000, set_theme_based_on_time, { ["repeat"] = -1 })
end, 1000)
```

## 🎛️ Workflow Customizations

### 1. **Project-Specific Settings**

**Create .nvimrc in project root:**
```lua
-- .nvimrc (in your project root)
vim.opt_local.shiftwidth = 4  -- 4 spaces for this project
vim.opt_local.expandtab = false  -- Use tabs instead of spaces

-- Project-specific keymaps
vim.keymap.set("n", "<leader>pr", "<cmd>!npm run build<cr>", { desc = "Build project" })
vim.keymap.set("n", "<leader>pt", "<cmd>!npm test<cr>", { desc = "Run tests" })

-- Set up project-specific LSP settings
require("lspconfig").ts_ls.setup({
  settings = {
    typescript = {
      preferences = {
        importModuleSpecifier = "relative"
      }
    }
  }
})
```

**Auto-load project settings:**
```lua
-- In lua/config/autocmds.lua:
vim.api.nvim_create_autocmd({"BufEnter", "BufWinEnter"}, {
  pattern = {"*"},
  callback = function()
    local nvimrc = vim.fn.findfile(".nvimrc", ".;")
    if nvimrc ~= "" then
      vim.cmd("source " .. nvimrc)
    end
  end
})
```

### 2. **Filetype-Specific Customizations**

**JavaScript/TypeScript specific:**
```lua
-- Create lua/ftplugin/javascript.lua
vim.opt_local.shiftwidth = 2
vim.opt_local.tabstop = 2

-- Enable format on save for JS files
vim.api.nvim_create_autocmd("BufWritePre", {
  buffer = 0,
  callback = function()
    vim.lsp.buf.format({ async = false })
  end
})

-- JS-specific keymaps
vim.keymap.set("n", "<leader>cl", "console.log()<Esc>F(a", { buffer = true, desc = "Insert console.log" })
```

**Ruby specific:**
```lua
-- Create lua/ftplugin/ruby.lua
vim.opt_local.shiftwidth = 2
vim.opt_local.tabstop = 2

-- Ruby-specific snippets
vim.keymap.set("n", "<leader>rb", "binding.pry<Esc>", { buffer = true, desc = "Insert binding.pry" })
```

## 🔌 Plugin Ecosystem Extensions

### 1. **Add Testing Support**

```lua
-- Add to lua/plugins/extras.lua:
{
  "nvim-neotest/neotest",
  dependencies = {
    "nvim-neotest/neotest-jest",
    "olimorris/neotest-rspec",
    "nvim-neotest/neotest-plenary",
  },
  config = function()
    require("neotest").setup({
      adapters = {
        require("neotest-jest")({
          jestCommand = "npm test --",
          env = { CI = true },
        }),
        require("neotest-rspec"),
      }
    })
  end,
  keys = {
    { "<leader>tr", function() require("neotest").run.run() end, desc = "Run nearest test" },
    { "<leader>tf", function() require("neotest").run.run(vim.fn.expand("%")) end, desc = "Run file tests" },
    { "<leader>ts", function() require("neotest").summary.toggle() end, desc = "Test summary" },
  }
}
```

### 2. **Add Debugging Support**

```lua
{
  "mfussenegger/nvim-dap",
  dependencies = {
    "rcarriga/nvim-dap-ui",
    "theHamsta/nvim-dap-virtual-text",
    "mxsdev/nvim-dap-vscode-js",
  },
  config = function()
    local dap = require("dap")
    local dapui = require("dapui")
    
    dapui.setup()
    require("nvim-dap-virtual-text").setup()
    
    -- Auto open/close DAP UI
    dap.listeners.after.event_initialized["dapui_config"] = dapui.open
    dap.listeners.before.event_terminated["dapui_config"] = dapui.close
  end,
  keys = {
    { "<leader>db", function() require("dap").toggle_breakpoint() end, desc = "Toggle breakpoint" },
    { "<leader>dc", function() require("dap").continue() end, desc = "Continue" },
    { "<leader>ds", function() require("dap").step_over() end, desc = "Step over" },
  }
}
```

## 🎓 Creating Your Own Plugin

### 1. **Simple Utility Plugin**

**Create lua/plugins/my-utils.lua:**
```lua
local M = {}

-- Quick project switcher
function M.switch_project()
  local projects = {
    ["~/work/frontend"] = "Frontend",
    ["~/work/backend"] = "Backend", 
    ["~/personal/blog"] = "Blog"
  }
  
  vim.ui.select(
    vim.tbl_values(projects),
    { prompt = "Select project:" },
    function(choice)
      for path, name in pairs(projects) do
        if name == choice then
          vim.cmd("cd " .. vim.fn.expand(path))
          vim.cmd("Telescope find_files")
          break
        end
      end
    end
  )
end

-- Quick note taking
function M.quick_note()
  local note = vim.fn.input("Quick note: ")
  if note ~= "" then
    local notes_file = vim.fn.expand("~/notes/quick.md")
    local timestamp = os.date("%Y-%m-%d %H:%M")
    local lines = { "", "## " .. timestamp, note, "" }
    
    vim.fn.writefile(lines, notes_file, "a")
    vim.notify("Note added!")
  end
end

return {
  -- Your custom plugin
  {
    dir = "",  -- Local plugin
    name = "my-utils",
    keys = {
      { "<leader>sp", function() M.switch_project() end, desc = "Switch Project" },
      { "<leader>qn", function() M.quick_note() end, desc = "Quick Note" },
    }
  }
}
```

## 💡 Pro Tips

### 1. **Use Local Configuration Files**

Create `lua/config/local.lua` (add to `.gitignore`):
```lua
-- Personal settings that shouldn't be committed
return {
  email = "your.email@example.com",
  work_directories = { "/work/project1", "/work/project2" },
  personal_theme = "catppuccin",
  enable_copilot = true,
}
```

Then in your main config:
```lua
local ok, local_config = pcall(require, "config.local")
if ok then
  -- Use local_config.email, etc.
end
```

### 2. **Environment-Based Configuration**

```lua
local function is_work_machine()
  return vim.fn.hostname():match("work%-") ~= nil
end

if is_work_machine() then
  -- Work-specific plugins and settings
else
  -- Personal machine settings
end
```

### 3. **Performance Monitoring**

```lua
-- Add to lua/config/autocmds.lua:
vim.api.nvim_create_user_command("StartupTime", function()
  vim.cmd("Lazy profile")
end, { desc = "Show startup time breakdown" })
```

**Next:** [Troubleshooting](05-troubleshooting.md) - Learn how to debug common issues.