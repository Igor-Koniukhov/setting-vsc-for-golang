# Configuring VSC for Working with Golang (making it similar and convenient like the Goland IDE).
### Installing:
- the official Go plugin from the Go Team at Google;
- the Code Runner plugin;
- the JetBrains Darcula Theme;
- the JetBrains Icon Theme;
- the JetBrains IDE Keymap;
- JSON to Go;
- vscode-go-syntax;

Open the Command Palette using Ctrl+Shift+P and type:
- "Go: Install/Update Tools,"

then select all the tools and install them.
To use the 'Go to Definition' functionality, it's necessary that the language server 'gopls' is installed and properly configured in your environment.

```bash
go install golang.org/x/tools/gopls@latest
```
Ensure that 'gopls' support is enabled in settings.json (go to Settings, type 'json' in the search bar and click 'Edit in settings.json.'):

```json
"go.useLanguageServer": true
```
For automatic package imports, 'goimports' should be installed if it's not already installed:

```bash
go install golang.org/x/tools/cmd/goimports@latest
```

### Configure Visual Studio Code to use 'goimports'.
Add the following lines for automatic formatting (if 'gofmt' is not installed then 'go install golang.org/x/tools/cmd/gofmt@latest') and organizing imports:

```json
"[go]": {
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "golang.go",
  "editor.codeActionsOnSave": {
      "source.organizeImports": true
  }
}
```
In the settings.json file, we can add additional settings for color highlighting of variables and types, for this, you need to add rules in settings.json; here is my example:


```json
"editor.tokenColorCustomizations": {
    "textMateRules": [
      {
        "scope": ["source.go storage.type"],
        "settings": {
          "foreground": "#ce720f"
        }
      },
      {
        "scope": "variable.other.declaration.go",
        "settings": {
          "foreground": "#bdb7b7"
        }
      },
      {
        "scope": "meta.section.struct.go",
        "settings": {
          "foreground": "#b2b890"
        }
      },
      {
        "scope": "entity.name.type.go",
        "settings": {
          "foreground": "#bdb7b7",
          "fontStyle": "bold"
        }
      },
      {
        "scope": "variable.other.assignment.go",
        "settings": {
          "foreground": "#bdb7b7"
        }
      },
      {
        "scope": "string.quoted.double.go",
        "settings": {
          "fontStyle": "italic"
        }
      }
    ]
  },
  ```
To precisely adjust the highlighting, you need to add the exact name of selectors; for this, you can use the theme inspector:
Ctrl+Shift+P and enter 'Developer: Inspect Editor Tokens and Scopes.' Click on a data type in the structure to see the exact selector.

To further resemble JetBrains Goland, you can add the following:
 
```json
"editor.fontSize": 15,
"editor.lineHeight": 25,
"editor.fontFamily": "Menlo",
```

If you want the font to change with Ctrl and the mouse wheel:


```json
"editor.minimap.enabled": false,
"editor.mouseWheelZoom": true,
```
To make a run-arrow button appear, which will run 'go run main.go,' the Code Runner plugin was installed, but to make this button work regardless of the location of 'main.go'
(since 'main.go' can be located in the 'cmd/web' directory and the 'main' package can include several files, for example, 'routes.go'), the following approach can be implemented (it works for me).

Create a 'run_go.sh' file in a global directory, for example, '/usr/local/bin'


```bash
#!/bin/bash

# Function to run Go files, excluding _test.go files
r#!/bin/bash

# Function to build and run Go files, excluding _test.go files
run_go() {
    project_root=$(git rev-parse --show-toplevel 2>/dev/null || echo "$PWD")
    project_name=$(basename "$project_root") # Use the project root directory name as the project name
    output_file="$project_root/$project_name"
    
    # Find Go files excluding _test.go files
    files=$(find . -name "*.go" ! -name "*_test.go")
    if [ -n "$files" ]; then
        # Build the project
        go build -o "$output_file" $files
        if [ $? -eq 0 ]; then
            # Run the built executable from the project root
            (cd "$project_root" && GO_ENV_FILE="$project_root/.env" "$output_file")
        else
            echo "Build failed"
            exit 1
        fi
    else
        echo "No Go files found to build"
        exit 1
    fi
}

# Find the main.go file
main_file=$(find . -name "main.go" | head -n 1)

if [ -n "$main_file" ]; then
    cd $(dirname "$main_file") && run_go
else
    echo "main.go not found"
    exit 1
fi

```
Make the script executable:

```bash
chmod +x /usr/local/bin/run_go.sh
```

Open Visual Studio Code settings (settings.json) and add or update the executorMap settings for Code Runner:

```json 
{
    "code-runner.executorMap": {
        "go": "/usr/local/bin/run_go.sh"
    },
    "code-runner.fileDirectoryAsCwd": true,
    "code-runner.saveAllFilesBeforeRun": true
}
```

Now your VSC editor will somewhat resemble the Goland IDE by JetBrains :)

These are all my settings in settings.json:

```json
{
  "workbench.iconTheme": "vscode-jetbrains-icon-theme",
  "workbench.colorTheme": "JetBrains Darcula Theme",
  "editor.wordWrap": "on",
  "editor.minimap.enabled": false,
  "editor.mouseWheelZoom": true,
  "editor.fontSize": 15,
  "editor.lineHeight": 25,
  "editor.fontFamily": "Menlo",
  "editor.tokenColorCustomizations": {
    "textMateRules": [
      {
        "scope": ["source.go storage.type"],
        "settings": {
          "foreground": "#ce720f"
        }
      },
      {
        "scope": "variable.other.declaration.go",
        "settings": {
          "foreground": "#bdb7b7"
        }
      },
      {
        "scope": "meta.section.struct.go",
        "settings": {
          "foreground": "#b2b890"
        }
      },
      {
        "scope": "entity.name.type.go",
        "settings": {
          "foreground": "#bdb7b7",
          "fontStyle": "bold"
        }
      },
      {
        "scope": "variable.other.assignment.go",
        "settings": {
          "foreground": "#bdb7b7"
        }
      },
      {
        "scope": "string.quoted.double.go",
        "settings": {
          "fontStyle": "italic"
        }
      }
    ]
  },

  "reactSnippets.settings.importReactOnTop": false,
  "[typescriptreact]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[typescript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[html]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[javascript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[go]": {
    "editor.formatOnSave": true,
    "editor.defaultFormatter": "golang.go",
    "editor.codeActionsOnSave": {
      "source.organizeImports": "explicit"
    }
  },

  "go.useLanguageServer": true,
  "go.testFlags": ["-v"],
  "go.lintOnSave": "file",
  "go.vetOnSave": "workspace",
  "go.coverOnSave": true,
  "go.coverageDecorator": {
    "type": "highlight"
  },
  "go.enableCodeLens": {
    "runtest": true
  },
  "go.formatTool": "gofmt",
  "go.addTags": {
    "tags": "json",
    "options": "json=omitempty",
    "promptForTags": false
  },
  "go.toolsManagement.autoUpdate": true,
  "go.testTimeout": "30s",
  "gopls": {
    "ui.completion.usePlaceholders": true,
    "verboseOutput": true
  },
  "files.autoSave": "afterDelay",
  "vs-kubernetes": {
    "vscode-kubernetes.helm-path-linux": "/home/koniukhov/.local/state/vs-kubernetes/tools/helm/linux-amd64/helm",
    "vscode-kubernetes.kubectl-path-linux": "/home/koniukhov/.local/state/vs-kubernetes/tools/kubectl/kubectl",
    "vscode-kubernetes.minikube-path-linux": "/home/koniukhov/.local/state/vs-kubernetes/tools/minikube/linux-amd64/minikube"
  },
  "editor.fontLigatures": false,
  "window.zoomLevel": 1.2,
  "terminal.integrated.mouseWheelZoom": true,
  "workbench.editor.enablePreview": false,
  "terminal.integrated.fontSize": 10,
  "terminal.integrated.fontFamily": "monospace",
  "[jsonc]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "editor.tabSize": 2,
  "editor.formatOnPaste": true,
  "json.schemas": [],
  "code-runner.executorMap": {
    "javascript": "node",
    "php": "php",
    "python": "python -u",
    "go": "/usr/local/bin/run_go.sh",
    "lua": "lua",
    "powershell": "powershell -ExecutionPolicy ByPass -File",
    "bat": "cmd /c",
    "shellscript": "bash",
    "typescript": "ts-node",
    "coffeescript": "coffee",
    "rust": "cd $dir && rustc $fileName && $dir$fileNameWithoutExt",
    "pascal": "cd $dir && fpc $fileName and $dir$fileNameWithoutExt",
    "sass": "sass --style expanded",
    "scss": "scss --style expanded",
    "less": "cd $dir and lessc $fileName $fileNameWithoutExt.css"
  },
  "code-runner.fileDirectoryAsCwd": true,
  "code-runner.saveAllFilesBeforeRun": true,
  "eslint.trace.server": "off"
}
```
