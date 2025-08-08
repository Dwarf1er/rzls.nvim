# Example configurations

This document contains configuration examples for:

[1. lazy.nvim](#1-lazynvim-example)</br>
[2. vim.pack (neovim 0.12+)](#2-vimpack-example-neovim-012)

## 1. lazy.nvim example

Example configuration of `seblyng/roslyn.nvim` with `lazy.nvim`

```lua
return {
    {
        "seblyng/roslyn.nvim",
        ft = { "cs", "razor" },
        dependencies = {
            {
                -- By loading as a dependencies, we ensure that we are available to set
                -- the handlers for Roslyn.
                "tris203/rzls.nvim",
                config = true,
            },
        },
        config = function()
            -- Use one of the methods in the Integration section to compose the command.
            local cmd = {}

            vim.lsp.config("roslyn", {
                cmd = cmd,
                handlers = require("rzls.roslyn_handlers"),
                settings = {
                    ["csharp|inlay_hints"] = {
                        csharp_enable_inlay_hints_for_implicit_object_creation = true,
                        csharp_enable_inlay_hints_for_implicit_variable_types = true,

                        csharp_enable_inlay_hints_for_lambda_parameter_types = true,
                        csharp_enable_inlay_hints_for_types = true,
                        dotnet_enable_inlay_hints_for_indexer_parameters = true,
                        dotnet_enable_inlay_hints_for_literal_parameters = true,
                        dotnet_enable_inlay_hints_for_object_creation_parameters = true,
                        dotnet_enable_inlay_hints_for_other_parameters = true,
                        dotnet_enable_inlay_hints_for_parameters = true,
                        dotnet_suppress_inlay_hints_for_parameters_that_differ_only_by_suffix = true,
                        dotnet_suppress_inlay_hints_for_parameters_that_match_argument_name = true,
                        dotnet_suppress_inlay_hints_for_parameters_that_match_method_intent = true,
                    },
                    ["csharp|code_lens"] = {
                        dotnet_enable_references_code_lens = true,
                    },
                },
            })
            vim.lsp.enable("roslyn")
        end,
        init = function()
            -- We add the Razor file types before the plugin loads.
            vim.filetype.add({
                extension = {
                    razor = "razor",
                    cshtml = "razor",
                },
            })
        end,
    },
}
```

## 2. vim.pack example (neovim 0.12+)

Example of a modular configuration with `vim.pack` in `neovim v0.12+`

### Overall structure

```text
├── init.lua
├── lsp
│   └── roslyn.lua
└── lua
    ├── config
    │   ├── filetypes.lua
    │   └── lsp.lua
    └── plugins
        ├── mason-nvim.lua
        ├── roslyn-nvim.lua
        └── rzls-nvim.lua
```

### `init.lua`
```lua
-- Core
require("config.filetypes")

-- Plugins
require("plugins.mason-nvim")
require("plugins.rzls-nvim")
require("plugins.roslyn-nvim")
require("plugins.nvim-treesitter")

-- LSP configuration
require("config.lsp")
```

### `lsp/roslyn.lua`

```lua
require("mason-registry")
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

vim.lsp.config("roslyn", {
    cmd = cmd,
    handlers = require("rzls.roslyn_handlers"),
    filetypes = { "cs" },
    root_markers = { { ".sln", ".csproj", "project.json" }, ".git" },
    settings = {
        ["csharp|inlay_hints"] = {
            csharp_enable_inlay_hints_for_implicit_object_creation = true,
            csharp_enable_inlay_hints_for_implicit_variable_types = true,

            csharp_enable_inlay_hints_for_lambda_parameter_types = true,
            csharp_enable_inlay_hints_for_types = true,
            dotnet_enable_inlay_hints_for_indexer_parameters = true,
            dotnet_enable_inlay_hints_for_literal_parameters = true,
            dotnet_enable_inlay_hints_for_object_creation_parameters = true,
            dotnet_enable_inlay_hints_for_other_parameters = true,
            dotnet_enable_inlay_hints_for_parameters = true,
            dotnet_suppress_inlay_hints_for_parameters_that_differ_only_by_suffix = true,
            dotnet_suppress_inlay_hints_for_parameters_that_match_argument_name = true,
            dotnet_suppress_inlay_hints_for_parameters_that_match_method_intent = true,
        },
        ["csharp|code_lens"] = {
            dotnet_enable_references_code_lens = true,
        },
        ["csharp|completion"] = {
            dotnet_show_name_completion_suggestions = true,
            dotnet_show_completion_items_from_unimported_namespaces = true,
        },
        ["csharp|background_analysis"] = {
            background_analysis = {
                dotnet_analyzer_diagnostics_scope = "fullSolution",
                dotnet_compiler_diagnostics_scope = "fullSolution",
            },
        },
    }
})
```

### `lua/config/filetypes.lua`

```lua
vim.filetype.add({
  extension = {
    razor = "razor",
    cshtml = "razor",
  },
})
```

### `lua/config/lsp.lua`

```lua
vim.api.nvim_create_autocmd('LspAttach', {
    callback = function(ev)
    local client = vim.lsp.get_client_by_id(ev.data.client_id)
    if client and client:supports_method(vim.lsp.protocol.Methods.textDocument_completion) then
        vim.opt.completeopt = { 'menu', 'menuone', 'noinsert', 'fuzzy', 'popup' }
        vim.lsp.completion.enable(true, client.id, ev.buf, { autotrigger = true })
        vim.keymap.set('i', '<C-Space>', function()
        vim.lsp.completion.get()
      end)
    end
  end,
})
```

### `lua/plugins/mason-nvim.lua`

```lua
vim.pack.add({
    {src = "https://github.com/neovim/nvim-lspconfig"},
    {src = "https://github.com/mason-org/mason.nvim.git"},
    {src = "https://github.com/mason-org/mason-lspconfig.nvim"},
    {src = "https://github.com/WhoIsSethDaniel/mason-tool-installer.nvim"},
})

require("mason").setup({
    registries = {
        "github:mason-org/mason-registry",
        "github:Crashdummyy/mason-registry"
    }
})

require("mason-lspconfig").setup()
require("mason-tool-installer").setup({
    ensure_installed = {
        "lua_ls",
        "html-lsp",
        "roslyn",
        "rzls"
    }
})
```

### `lua/plugins/roslyn-nvim.lua`

```lua
vim.pack.add({ "https://github.com/seblyng/roslyn.nvim.git" })

require("roslyn").setup({
    ft = { "cs", "razor" }
})
```

### `lua/plugins/rzls-nvim.lua`

```lua
vim.pack.add({ "https://github.com/tris203/rzls.nvim.git" })

require("rzls").setup({
    path = "rzls" or nil
})
```