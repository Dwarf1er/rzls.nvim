# rzls.nvim 🚀

## Description 📄

`rzls.nvim` is a Neovim plugin for the Razor Language Server (rzls). It provides
language server protocol (LSP) support for Razor/Blazor/CSHTML files, bringing
powerful development features to Neovim. ✨

With `rzls.nvim`, you can enjoy a seamless coding experience with features like
auto-completion, go-to-definition, and more all from within Neovim 💻🔧

### Features

| Feature               | Support |
| --------------------- | ------- |
| Hover                 | ✅       |
| Diagnostics           | ✅       |
| Go To Definition      | ✅       |
| Go To References      | ✅       |
| Semantic Highlighting | ✅       |
| Formatting            | ✅       |
| Rename Symbol         | ✅       |
| Signature Help        | ✅       |
| Completions           | ✅       |
| Inlay Hints           | ✅       |
| Code Actions          | ✅       |
| Folding               | ✅       |
| CodeLens              | ❌       |
| Format New Files      | ❌       |

> [!NOTE]
> Semantic highlight groups need more configuration If you find a
> property that isn't highlighted properly and is identified with `:Inspect`
> please raise an issue, or a PR to link it to a highlight group.
 
## Table of Contents
- [rzls.nvim 🚀](#rzlsnvim-)
  - [Description 📄](#description-)
    - [Features](#features)
  - [Table of Contents](#table-of-contents)
  - [Dependencies](#dependencies)
  - [Installing the Dependencies](#installing-the-dependencies)
    - [roslyn](#roslyn)
      - [Mason](#mason)
      - [Manual](#manual)
    - [rzls](#rzls)
      - [Mason](#mason-1)
      - [Manual](#manual-1)
    - [html-lsp](#html-lsp)
      - [Mason](#mason-2)
      - [Manual](#manual-2)
    - [seblyng/roslyn.nvim](#seblyngroslynnvim)
      - [lazy.nvim](#lazynvim)
      - [vim.pack (neovim v0.12+)](#vimpack-neovim-v012)
  - [Installing `rzls.nvim`](#installing-rzlsnvim)
      - [lazy.nvim](#lazynvim-1)
      - [vim.pack (neovim v0.12+)](#vimpack-neovim-v012-1)
  - [Configuring Plugins](#configuring-plugins)
    - [Configuring `seblyng/roslyn.nvim`](#configuring-seblyngroslynnvim)
      - [Mason](#mason-3)
      - [Manually](#manually)
    - [Configuring `rzls.nvim`](#configuring-rzlsnvim)
  - [Additional Configuration](#additional-configuration)
    - [Telescope.nvim](#telescopenvim)
    - [Trouble.nvim](#troublenvim)
  - [Example Configuration](#example-configuration)
  - [Known Issues](#known-issues)
  - [Contributing](#contributing)
  - [Helping Out](#helping-out)
  - [License](#license)

## Dependencies

Before installing `rzls.nvim`, you must have installed the following:

- [`roslyn`](https://github.com/crashdummyy/roslynlanguageserver) - The C# language server. Required for all C# and Razor/Blazor integrations.<br/>
    [-> Jump to installation instructions](#roslyn)
- [`rzls`](https://github.com/crashdummyy/rzls) - The Razor language server. Handles Razor/Blazor/CSHTML integrations.<br/>
    [-> Jump to installation instructions](#rzls)
- [`html-lsp`](https://github.com/microsoft/vscode-html-languageservice) - The HTML language server. Provides completions and formatting for HTML inside of `.razor` files.<br/>
    [-> Jump to installation instructions](#html-lsp)
- [`seblyng/roslyn.nvim`](https://github.com/seblyng/roslyn.nvim) - Neovim integration for Roslyn. Handles communication between Neovim and the Roslyn language server.<br/>
    [-> Jump to installation instructions](#seblyngroslynnvim)

> [!CAUTION]
> Please see the [configuring seblyng/roslyn.nvim](#configuring-seblyngroslynnvim) section for extra arguments that must be passed to
> `roslyn.nvim` setup.

## Installing the Dependencies

A note on `Mason`: all mason-based installs will require the following setup in your config **once**, everything else will be added to the `ensure_installed` list:
```lua
require("mason").setup({
    registries = {
        "github:mason-org/mason-registry",
        "github:Crashdummyy/mason-registry"
    }
})

require("mason-lspconfig").setup()
require("mason-tool-installer").setup({
    ensure_installed = {
        "roslyn",
        "rzls",
        "html-lsp"
    }
})
```

### roslyn 

#### Mason

Add `"roslyn"` to your mason-tool-installer `ensure_installed` list.

#### Manual

1. Go to the official `roslyn` feed [here](https://dev.azure.com/azure-public/vside/_artifacts/feed/vs-impl)
2. Download the `Microsoft.CodeAnalysis.LanguageServer` artifact for your system
3. Unzip the downloaded artifact to `$directory-of-your-choice`
4. Use `$directory-of-your-choice` in the `cmd` value to point to the `Microsoft.CodeAnalysis.LanguageServer.dll` when [configuring roslyn.nvim](#configuring-roslynnvim)

### rzls

#### Mason

Add `"rzls"` to your mason-tool-installer `ensure_installed` list.

#### Manual

### html-lsp

#### Mason

Add `"html-lsp"` to your mason-tool-installer `ensure_installed` list.

#### Manual

Using [`npm`](https://github.com/npm/cli) run the following command:
```bash
npm install --save vscode-html-languageservice
```

### seblyng/roslyn.nvim

No matter which package manager you use, make sure that you configure [`seblyng/roslyn.nvim`](#configuring-seblyngroslynnvim)

#### lazy.nvim

```lua
return {
    "seblyng/roslyn.nvim"
}
```

#### vim.pack (neovim v0.12+)

```lua
vim.pack.add({ "https://github.com/seblyng/roslyn.nvim.git" })
```

## Installing `rzls.nvim`

#### lazy.nvim

```lua
return {
    "tris203/rzls.nvim"
}
```

#### vim.pack (neovim v0.12+)

```lua
vim.pack.add({ "https://github.com/tris203/rzls.nvim.git" })
```

## Configuring Plugins

### Configuring `seblyng/roslyn.nvim` 

To ensure seamless communication between roslyn.nvim and rzls, you need to configure roslyn.nvim with specific command-line arguments and handlers provided by rzls.nvim.

This involves composing the Roslyn language server command with arguments that point to the installed rzls components, whether you installed them manually or via Mason. You’ll also need to pass the handler functions defined in the rzls.roslyn_handlers module to roslyn.nvim’s setup.

Below, we provide examples of how to assemble this command and configure roslyn.nvim accordingly based on your installation method.

#### Mason

```lua
local mason_registry = require("mason-registry")

local rzls_path = vim.fn.expand("$MASON/packages/rzls/libexec")
local cmd = {
    "roslyn",
    "--stdio",
    "--logLevel=Information",
    "--extensionLogDirectory=" .. vim.fs.dirname(vim.lsp.get_log_path()),
    "--razorSourceGenerator=" .. vim.fs.joinpath(rzls_path, "Microsoft.CodeAnalysis.Razor.Compiler.dll"),
    "--razorDesignTimePath=" .. vim.fs.joinpath(rzls_path, "Targets", "Microsoft.NET.Sdk.Razor.DesignTime.targets"),
    "--extension",
    vim.fs.joinpath(rzls_path, "RazorExtension", "Microsoft.VisualStudioCode.RazorExtension.dll"),
}

require("roslyn").setup({
    cmd = cmd,
    config = {
        -- The rest of your Roslyn configuration
        handlers = require("rzls.roslyn_handlers"),
    },
})
```

#### Manually

```lua
-- Adjust these paths to where you installed Roslyn and rzls.
local roslyn_base_path = vim.fs.joinpath(vim.fn.stdpath("data"), "roslyn")
local rzls_base_path = vim.fs.joinpath(vim.fn.stdpath("data"), "rzls")

local cmd = {
    "dotnet",
    vim.fs.joinpath(roslyn_base_path, "Microsoft.CodeAnalysis.LanguageServer.dll"),
    "--stdio",
    "--logLevel=Information",
    "--extensionLogDirectory=" .. vim.fs.dirname(vim.lsp.get_log_path()),
    "--razorSourceGenerator=" .. vim.fs.joinpath(rzls_base_path, "Microsoft.CodeAnalysis.Razor.Compiler.dll"),
    "--razorDesignTimePath="
        .. vim.fs.joinpath(rzls_base_path, "Targets", "Microsoft.NET.Sdk.Razor.DesignTime.targets"),
}

require("roslyn").setup({
    cmd = cmd,
    config = {
        -- The rest of your Roslyn configuration
        handlers = require("rzls.roslyn_handlers"),
    },
})
```

### Configuring `rzls.nvim`

You can customize `rzls.nvim` by passing a configuration table to its `setup` function. Here are the options:

- **`capabilities`**  
  A table describing what features your LSP client supports (like completion, hover, etc).  
  If you're using a completion plugin (like `nvim-cmp`), you can pass its capabilities here.
  If you're unsure, you can leave this out or consult the documentation of your completion provider.

- **`path`**  
  The file system path to the `rzls` executable.  
  If you installed `rzls` via Mason, you don't need to set this.  
  But if you installed it manually, set this to the full path to your `rzls` binary.

- **`on_attach`**  
  A function called when the language server attaches to a buffer, often used to set up keymaps or other buffer-local settings.  
  If you’re handling `on_attach` globally via autocommands or elsewhere, you can omit this or pass an empty function.

```lua
require("rzls").setup()
```

## Additional Configuration

### Telescope.nvim

If you use [`telescope.nvim`](https://github.com/nvim-telescope/telescope.nvim)
for definitions and references then you may want to add additional filtering
to exclude references in the generated virtual files.

```lua
require("telescope").setup({
    defaults = {
        file_ignore_patterns = { "%__virtual.cs$" },
    },
})
```

### Trouble.nvim

If you use [`trouble.nvim`](https://github.com/folke/trouble.nvim) for
diagnostics, then you want to exclude the virtual buffers from diagnostics

```lua
require("trouble").setup({
    modes = {
        diagnostics = {
            filter = function(items)
                return vim.tbl_filter(function(item)
                    return not string.match(item.basename, [[%__virtual.cs$]])
                end, items)
            end,
        },
    },
})
```

## Example Configuration

For detailed configuration examples, see our [example configuration](./doc/CONFIGURATION.md)

## Known Issues

- Native Windows support doesn't work at this time, due to path normalization.
- Opening a CS file first means that Roslyn and rzls don't connect properly.

## Contributing

This plugin is still under construction. The Razor Language Server (rzls) uses a
variety of custom methods that need to be understood and implemented. We are
actively working on this and appreciate your patience.

We welcome contributions from the community for support of new features, fixing
bugs, issues or things on the TODO-list (Grep the code for `TODO`). If you have
experience with LSP or
Razor and would like to contribute, please open a Pull Request. If
you encounter any issues or have suggestions for improvements, please open an
Issue. Your input is valuable in making this plugin more robust and efficient.

There is a discord community linked in the discussion below, if you have
anything you would like to discuss, or you want to help. Come say hi.

## Helping Out

If you want to help out, then please see the discussion here, and leave a
comment with your details in this [discussion](https://github.com/tris203/rzls.nvim/discussions/1).

## License

This project is licensed under the [MIT license](LICENSE).