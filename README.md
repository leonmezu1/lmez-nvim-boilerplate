# 🚀 LMEZ Neovim Boilerplate

A modern, feature-rich Neovim configuration designed for React, Rails, and full-stack development.

## ✨ Features

- **🎨 Beautiful UI** - Monokai Pro theme with modern statusline
- **🔍 Fuzzy Finding** - Telescope for files, text, git, and more
- **📁 File Explorer** - Neo-tree with Git integration
- **💡 Smart Completion** - LSP-powered autocompletion
- **🌳 Syntax Highlighting** - TreeSitter for accurate parsing
- **🔧 Language Servers** - Mason for easy LSP management
- **⚡ Fast Startup** - Lazy loading for optimal performance
- **🎯 React/Rails Ready** - Pre-configured for modern web development

## 🚀 Quick Start

```bash
# Clone and run the setup script
git clone <your-repo-url>
cd lmez-nvim-boilerplate
chmod +x setup_nvim.sh
./setup_nvim.sh
```

Then launch Neovim and let the magic happen! 🪄

## 📚 Documentation

This configuration is fully documented to help you understand every piece:

### Core Documentation
- **[Architecture & Structure](docs/01-architecture.md)** - How the config is organized
- **[Plugin Breakdown](docs/02-plugins.md)** - What each plugin does and why
- **[Configuration Patterns](docs/03-patterns.md)** - Common patterns and best practices

### Customization Guides
- **[Customization Guide](docs/04-customization.md)** - Make it your own
- **[Troubleshooting](docs/05-troubleshooting.md)** - Common issues and solutions

### Learning Path
- **[Building Your Own](docs/06-build-your-own.md)** - Step-by-step guide to create from scratch

## 🎯 Key Bindings (Leader = Space)

| Key | Action | Description |
|-----|--------|-------------|
| `<leader>ff` | Find Files | Search files in project |
| `<leader>fg` | Live Grep | Search text in project |
| `<leader>e` | File Explorer | Toggle Neo-tree |
| `<leader>ca` | Code Actions | LSP code actions |
| `<leader>rn` | Rename | LSP rename symbol |
| `gd` | Go to Definition | Jump to definition |
| `K` | Hover | Show documentation |

## 🛠 Tech Stack

- **Base**: Neovim 0.8+
- **Plugin Manager**: Lazy.nvim
- **Language Protocol**: Built-in LSP + Mason
- **Completion**: nvim-cmp
- **Syntax**: TreeSitter
- **Finder**: Telescope
- **Explorer**: Neo-tree
- **Theme**: Monokai Pro

## 🤝 Learning Philosophy

This configuration is designed to be:
- **Educational** - Every choice is explained
- **Modular** - Easy to modify and extend
- **Transparent** - No magic, everything is visible
- **Practical** - Real-world development ready

## 📖 Learning Resources

- Read the documentation in order starting with [Architecture](docs/01-architecture.md)
- Experiment with one plugin at a time
- Check `:help` for Neovim built-in documentation
- Join the community at [r/neovim](https://reddit.com/r/neovim)

---

**Happy coding!** 🎉

> "The best way to learn is to build it yourself" - This boilerplate gives you the knowledge to do exactly that.