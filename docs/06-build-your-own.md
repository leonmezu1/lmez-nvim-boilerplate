# 🏗️ Building Your Own Configuration

This guide walks you through building a Neovim configuration from scratch. By the end, you'll understand every line and can confidently maintain and extend it.

## 🎯 Learning Approach

### Why Build From Scratch?
- **Deep Understanding**: Know every plugin and setting
- **No Bloat**: Only what you actually use
- **Personal Ownership**: Truly make it yours
- **Debugging Skills**: Fix issues with confidence

### The Process
1. Start with absolute basics
2. Add one feature at a time
3. Test thoroughly before moving on
4. Document your choices

## 🏁 Phase 1: Foundation (10 minutes)

### Step 1: Basic Structure
```bash
# Clean slate
rm -rf ~/.config/nvim
mkdir -p ~/.config/nvim/lua/config
```

### Step 2: Minimal init.lua
```lua
-- ~/.config/nvim/init.lua
vim.g.mapleader = " "
vim.g.maplocalleader = " "

-- Load core configuration
require("config.options")
require("config.keymaps")

print("Neovim config loaded!")
```

### Step 3: Essential Options
```lua
-- ~/.config/nvim/lua/config/options.lua
local opt = vim.opt

-- Line numbers
opt.number = true
opt.relativenumber = true

-- Indentation
opt.expandtab = true
opt.shiftwidth = 2
opt.tabstop = 2
opt.smartindent = true

-- Search
opt.ignorecase = true
opt.smartcase = true
opt.hlsearch = false

-- UI
opt.termguicolors = true
opt.signcolumn = "yes"
opt.cursorline = true
opt.mouse = "a"

-- Behavior
opt.scrolloff = 8
opt.updatetime = 250

print("Options loaded")
```

### Step 4: Basic Keymaps
```lua
-- ~/.config/nvim/lua/config/keymaps.lua
local map = vim.keymap.set

-- Better navigation
map("n", "<C-h>", "<C-w>h", { desc = "Window left" })
map("n", "<C-j>", "<C-w>j", { desc = "Window down" })
map("n", "<C-k>", "<C-w>k", { desc = "Window up" })
map("n", "<C-l>", "<C-w>l", { desc = "Window right" })

-- Better indenting
map("v", "<", "<gv")
map("v", ">", ">gv")

-- Move lines
map("n", "<A-j>", ":m .+1<cr>==", { desc = "Move line down" })
map("n", "<A-k>", ":m .-2<cr>==", { desc = "Move line up" })
map("v", "<A-j>", ":m '>+1<cr>gv=gv", { desc = "Move lines down" })
map("v", "<A-k>", ":m '<-2<cr>gv=gv", { desc = "Move lines up" })

-- Quick save
map("n", "<C-s>", "<cmd>w<cr>", { desc = "Save file" })

print("Keymaps loaded")
```

### Test Phase 1
```bash
nvim
# You should see:
# - Line numbers
# - Syntax highlighting works
# - Cursor shows current line
# - Basic keybindings work
```

## 🔌 Phase 2: Plugin Manager (15 minutes)

### Step 1: Add Lazy.nvim Bootstrap
```lua
-- Add to ~/.config/nvim/init.lua (after leader keys)
local lazypath = vim.fn.stdpath("data") .. "/lazy/lazy.nvim"
if not vim.loop.fs_stat(lazypath) then
  print("Installing lazy.nvim...")
  vim.fn.system({
    "git",
    "clone",
    "--filter=blob:none",
    "https://github.com/folke/lazy.nvim.git",
    "--branch=stable",
    lazypath,
  })
end
vim.opt.rtp:prepend(lazypath)

-- Setup plugins
require("lazy").setup("plugins", {
  change_detection = { notify = false },
})
```

### Step 2: Create Plugin Directory
```bash
mkdir -p ~/.config/nvim/lua/plugins
```

### Step 3: First Plugin - Colorscheme
```lua
-- ~/.config/nvim/lua/plugins/colorscheme.lua
return {
  "folke/tokyonight.nvim",
  priority = 1000,  -- Load first
  config = function()
    require("tokyonight").setup({
      style = "storm",
      transparent = false,
    })
    vim.cmd.colorscheme("tokyonight")
  end,
}
```

### Test Phase 2
```bash
nvim
# First time will install plugins
# You should see Tokyo Night theme
# Run :Lazy to see plugin manager
```

## 🔍 Phase 3: File Navigation (20 minutes)

### Step 1: Add Telescope
```lua
-- ~/.config/nvim/lua/plugins/telescope.lua
return {
  "nvim-telescope/telescope.nvim",
  dependencies = { 
    "nvim-lua/plenary.nvim",
    {
      "nvim-telescope/telescope-fzf-native.nvim",
      build = "make",
      cond = function()
        return vim.fn.executable("make") == 1
      end,
    },
  },
  keys = {
    { "<leader>ff", "<cmd>Telescope find_files<cr>", desc = "Find Files" },
    { "<leader>fg", "<cmd>Telescope live_grep<cr>", desc = "Live Grep" },
    { "<leader>fb", "<cmd>Telescope buffers<cr>", desc = "Buffers" },
    { "<leader>fh", "<cmd>Telescope help_tags<cr>", desc = "Help Tags" },
  },
  config = function()
    local telescope = require("telescope")
    telescope.setup({
      defaults = {
        prompt_prefix = " ",
        selection_caret = " ",
      },
    })
    
    -- Load extensions
    pcall(telescope.load_extension, "fzf")
  end,
}
```

### Step 2: Add File Explorer
```lua
-- ~/.config/nvim/lua/plugins/file-explorer.lua
return {
  "nvim-neo-tree/neo-tree.nvim",
  branch = "v3.x",
  dependencies = {
    "nvim-lua/plenary.nvim",
    "nvim-tree/nvim-web-devicons",
    "MunifTanjim/nui.nvim",
  },
  keys = {
    { "<leader>e", "<cmd>Neotree toggle<cr>", desc = "File Explorer" },
  },
  config = function()
    require("neo-tree").setup({
      filesystem = {
        follow_current_file = {
          enabled = true,
        },
        use_libuv_file_watcher = true,
      },
    })
  end,
}
```

### Test Phase 3
```bash
nvim
# Try:
# <space>ff - Find files
# <space>fg - Search text (needs ripgrep: brew install ripgrep)
# <space>e  - File explorer
```

## 🌳 Phase 4: Syntax Highlighting (15 minutes)

### Step 1: Add TreeSitter
```lua
-- ~/.config/nvim/lua/plugins/treesitter.lua
return {
  "nvim-treesitter/nvim-treesitter",
  build = ":TSUpdate",
  event = { "BufReadPost", "BufNewFile" },
  dependencies = {
    "nvim-treesitter/nvim-treesitter-textobjects",
  },
  config = function()
    require("nvim-treesitter.configs").setup({
      ensure_installed = {
        "lua",
        "javascript",
        "typescript",
        "tsx",
        "json",
        "html",
        "css",
        "ruby",
        "markdown",
      },
      
      highlight = {
        enable = true,
      },
      
      indent = {
        enable = true,
      },
      
      -- Text objects
      textobjects = {
        select = {
          enable = true,
          lookahead = true,
          keymaps = {
            ["af"] = "@function.outer",
            ["if"] = "@function.inner",
            ["ac"] = "@class.outer",
            ["ic"] = "@class.inner",
          },
        },
      },
    })
  end,
}
```

### Test Phase 4
```bash
nvim some-code-file.js
# You should see much better syntax highlighting
# Try text objects: "vif" to select inside function
```

## 💡 Phase 5: LSP & Completion (25 minutes)

### Step 1: Add LSP Foundation
```lua
-- ~/.config/nvim/lua/plugins/lsp.lua
return {
  -- LSP Configuration
  {
    "neovim/nvim-lspconfig",
    event = { "BufReadPre", "BufNewFile" },
    dependencies = {
      "williamboman/mason.nvim",
      "williamboman/mason-lspconfig.nvim",
    },
    config = function()
      -- Setup Mason first
      require("mason").setup()
      
      -- Basic LSP keymaps
      vim.api.nvim_create_autocmd("LspAttach", {
        callback = function(args)
          local buffer = args.buf
          local function map(mode, key, action, desc)
            vim.keymap.set(mode, key, action, { buffer = buffer, desc = desc })
          end
          
          map("n", "gd", vim.lsp.buf.definition, "Go to Definition")
          map("n", "K", vim.lsp.buf.hover, "Hover Documentation")
          map("n", "<leader>ca", vim.lsp.buf.code_action, "Code Actions")
          map("n", "<leader>rn", vim.lsp.buf.rename, "Rename")
        end,
      })
      
      -- Setup language servers
      local lspconfig = require("lspconfig")
      local capabilities = require("cmp_nvim_lsp").default_capabilities()
      
      -- Lua
      lspconfig.lua_ls.setup({
        capabilities = capabilities,
        settings = {
          Lua = {
            diagnostics = {
              globals = { "vim" },
            },
          },
        },
      })
      
      -- TypeScript/JavaScript
      lspconfig.ts_ls.setup({
        capabilities = capabilities,
      })
      
      -- Add more servers as needed
    end,
  },

  -- Mason (LSP installer)
  {
    "williamboman/mason.nvim",
    cmd = "Mason",
    keys = {
      { "<leader>cm", "<cmd>Mason<cr>", desc = "Mason" },
    },
  },
}
```

### Step 2: Add Completion
```lua
-- ~/.config/nvim/lua/plugins/completion.lua
return {
  "hrsh7th/nvim-cmp",
  event = "InsertEnter",
  dependencies = {
    "hrsh7th/cmp-nvim-lsp",
    "hrsh7th/cmp-buffer",
    "hrsh7th/cmp-path",
    {
      "L3MON4D3/LuaSnip",
      dependencies = { "rafamadriz/friendly-snippets" },
      config = function()
        require("luasnip.loaders.from_vscode").lazy_load()
      end,
    },
    "saadparwaiz1/cmp_luasnip",
  },
  config = function()
    local cmp = require("cmp")
    local luasnip = require("luasnip")
    
    cmp.setup({
      snippet = {
        expand = function(args)
          luasnip.lsp_expand(args.body)
        end,
      },
      
      mapping = {
        ["<C-n>"] = cmp.mapping.select_next_item(),
        ["<C-p>"] = cmp.mapping.select_prev_item(),
        ["<C-d>"] = cmp.mapping.scroll_docs(-4),
        ["<C-f>"] = cmp.mapping.scroll_docs(4),
        ["<C-Space>"] = cmp.mapping.complete(),
        ["<C-e>"] = cmp.mapping.close(),
        ["<CR>"] = cmp.mapping.confirm({
          behavior = cmp.ConfirmBehavior.Replace,
          select = true,
        }),
        ["<Tab>"] = cmp.mapping(function(fallback)
          if cmp.visible() then
            cmp.select_next_item()
          elseif luasnip.expand_or_jumpable() then
            luasnip.expand_or_jump()
          else
            fallback()
          end
        end, { "i", "s" }),
      },
      
      sources = {
        { name = "nvim_lsp" },
        { name = "luasnip" },
        { name = "buffer" },
        { name = "path" },
      },
    })
  end,
}
```

### Test Phase 5
```bash
nvim test.js
# Install language servers: :Mason
# Install ts_ls and lua_ls
# Try:
# - Type something and see completion
# - Hover with K
# - Go to definition with gd
```

## 🎨 Phase 6: UI Enhancements (15 minutes)

### Step 1: Add Statusline
```lua
-- ~/.config/nvim/lua/plugins/ui.lua
return {
  -- Statusline
  {
    "nvim-lualine/lualine.nvim",
    event = "VeryLazy",
    dependencies = { "nvim-tree/nvim-web-devicons" },
    config = function()
      require("lualine").setup({
        options = {
          theme = "auto",
          globalstatus = true,
        },
        sections = {
          lualine_a = { "mode" },
          lualine_b = { "branch", "diff", "diagnostics" },
          lualine_c = { "filename" },
          lualine_x = { "encoding", "fileformat", "filetype" },
          lualine_y = { "progress" },
          lualine_z = { "location" },
        },
      })
    end,
  },

  -- Better notifications
  {
    "rcarriga/nvim-notify",
    keys = {
      {
        "<leader>un",
        function()
          require("notify").dismiss({ silent = true, pending = true })
        end,
        desc = "Dismiss notifications",
      },
    },
    opts = {
      timeout = 3000,
    },
    init = function()
      vim.notify = require("notify")
    end,
  },

  -- Icons
  { "nvim-tree/nvim-web-devicons", lazy = true },
}
```

### Step 2: Add Which-Key (Help System)
```lua
-- Add to ui.lua
{
  "folke/which-key.nvim",
  event = "VeryLazy",
  config = function()
    local wk = require("which-key")
    wk.setup()
    
    -- Register key groups
    wk.add({
      { "<leader>f", group = "file" },
      { "<leader>c", group = "code" },
      { "<leader>g", group = "git" },
    })
  end,
}
```

### Test Phase 6
```bash
nvim
# You should see:
# - Nice statusline at bottom
# - Better notifications
# - Press <space> and pause to see which-key help
```

## 🔄 Phase 7: Git Integration (10 minutes)

### Step 1: Add Git Signs
```lua
-- ~/.config/nvim/lua/plugins/git.lua
return {
  "lewis6991/gitsigns.nvim",
  event = { "BufReadPre", "BufNewFile" },
  config = function()
    require("gitsigns").setup({
      signs = {
        add = { text = "+" },
        change = { text = "~" },
        delete = { text = "_" },
        topdelete = { text = "‾" },
        changedelete = { text = "~" },
      },
      
      on_attach = function(buffer)
        local gs = package.loaded.gitsigns
        
        local function map(mode, l, r, desc)
          vim.keymap.set(mode, l, r, { buffer = buffer, desc = desc })
        end
        
        -- Navigation
        map("n", "]c", gs.next_hunk, "Next hunk")
        map("n", "[c", gs.prev_hunk, "Prev hunk")
        
        -- Actions
        map("n", "<leader>hs", gs.stage_hunk, "Stage hunk")
        map("n", "<leader>hr", gs.reset_hunk, "Reset hunk")
        map("n", "<leader>hp", gs.preview_hunk, "Preview hunk")
        map("n", "<leader>hb", gs.blame_line, "Blame line")
      end,
    })
  end,
}
```

### Test Phase 7
```bash
# In a git repository:
nvim
# You should see git status in the gutter
# Make changes and see the indicators
```

## 📝 Phase 8: Autocommands & Polish (10 minutes)

### Step 1: Add Autocommands
```lua
-- ~/.config/nvim/lua/config/autocmds.lua
local function augroup(name)
  return vim.api.nvim_create_augroup("config_" .. name, { clear = true })
end

-- Highlight on yank
vim.api.nvim_create_autocmd("TextYankPost", {
  group = augroup("highlight_yank"),
  callback = function()
    vim.highlight.on_yank()
  end,
})

-- Go to last location when opening a buffer
vim.api.nvim_create_autocmd("BufReadPost", {
  group = augroup("last_loc"),
  callback = function()
    local mark = vim.api.nvim_buf_get_mark(0, '"')
    local lcount = vim.api.nvim_buf_line_count(0)
    if mark[1] > 0 and mark[1] <= lcount then
      pcall(vim.api.nvim_win_set_cursor, 0, mark)
    end
  end,
})

-- Close some filetypes with <q>
vim.api.nvim_create_autocmd("FileType", {
  group = augroup("close_with_q"),
  pattern = { "qf", "help", "man", "lspinfo" },
  callback = function(event)
    vim.bo[event.buf].buflisted = false
    vim.keymap.set("n", "q", "<cmd>close<cr>", { buffer = event.buf, silent = true })
  end,
})
```

### Step 2: Load Autocommands
```lua
-- Add to init.lua
require("config.autocmds")
```

## 🎉 Final Configuration

Your complete structure should look like:
```
~/.config/nvim/
├── init.lua
└── lua/
    ├── config/
    │   ├── options.lua
    │   ├── keymaps.lua
    │   └── autocmds.lua
    └── plugins/
        ├── colorscheme.lua
        ├── telescope.lua
        ├── file-explorer.lua
        ├── treesitter.lua
        ├── lsp.lua
        ├── completion.lua
        ├── ui.lua
        └── git.lua
```

## 🚀 Next Steps

### Immediate Improvements
1. **Add more language servers** via `:Mason`
2. **Install ripgrep** for better search: `brew install ripgrep`
3. **Add a terminal plugin** like toggleterm
4. **Add formatting** with conform.nvim or null-ls

### Customization Ideas
1. **Change colorscheme** in `colorscheme.lua`
2. **Add more keybindings** in `keymaps.lua`
3. **Install language-specific plugins**
4. **Add debugging support** with nvim-dap

### Learning Resources
1. **`:help`** - Built-in documentation
2. **Plugin READMEs** - Each plugin has great docs
3. **`:Lazy`** - Explore available plugins
4. **Community configs** - Study others' setups

## 💡 Pro Tips

### 1. **One Plugin at a Time**
Don't add everything at once. Add, test, understand, then move on.

### 2. **Read Plugin Docs**
Every plugin has a README with examples and configuration options.

### 3. **Use :checkhealth**
Regular health checks help catch issues early.

### 4. **Version Control Your Config**
```bash
cd ~/.config/nvim
git init
git add .
git commit -m "Initial Neovim config"
```

### 5. **Keep Notes**
Document why you made certain choices. Future you will thank you.

## 🎯 You Did It!

Congratulations! You now have a solid understanding of:
- How Neovim configuration works
- Plugin management with Lazy.nvim
- LSP setup and troubleshooting  
- Key binding patterns
- Configuration organization

Your config is now truly **yours** - you understand every piece and can modify it with confidence.

Keep experimenting, keep learning, and most importantly, keep coding! 🚀