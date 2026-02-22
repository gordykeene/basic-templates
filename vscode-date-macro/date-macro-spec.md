<!-- markdownlint-disable MD007 MD009 MD010 MD029 MD032 MD034 -->

# Insert Data Macro

## Goal

Create a macro for VSCode that will work across all of my VSCode instances and repositories.

## Requirements

* Adding it to VS Code once per machine should be sufficient. I don't want to have to re-create/install it for every repository. 
* It should be able to be bound to a the `Ctrl+Alt+D` key-chord.

## Actions

When triggered, it will do the following things:

* Beginning from the top of the file, it will search for a line that contains `## Dailies` (case insensitive)
  * If `## Dailies` is not found, leave the cursor at the top of the file, and stop. No need to notify the user or report any error.
* From there it will search for the very next line beginning with `### ` (three "#" followed by at least one space).
  * If the `### ` marker is not found, then return to the `## Dailies` line, insert two blank line after it, and continue on to the next step (see [Missing Any Dates](#missing-any-dates) example).
  * Otherwise, beginning at the found line (and before the existing `### `)
* It will insert:
  * a new `### `
  * followed by the current date in YYYY-MM-DD-DDD format,
  * followed by a trailing new line (for reference only we will call this "new-line-1"),
  * followed by three more blank lines, (for reference only we will call these "new-line-2", "new-line-3", and "new-line-4"),
  * then position the cursor at the beginning of "new-line-3".

### Example(s)

Note in the following examples:
* The `|<-- cursor here` is not actually inserted, and just shown for explanation.
* The `{eof}` means the end of the file, and just shown for explanation.

#### Happy Path Example

Beginning with:

```md
# My Project Title

... some content

## Purpose

... some content

## Dailies

### 2001-09-18-Tue

... some content
```

running the macro would, for example, update it to the following. 

```md
# My Project Title

... some content

## Purpose

... some content

## Dailies

### 2026-02-21-Sat

|<-- cursor here

### 2001-09-18-Tue

... some content
```

#### Missing Any Dates

Beginning with:

```md
# My Project Title

... some content

## Dailies

... some content with out any lines starting with `### `
```

The macro would result in:

```md
# My Project Title

... some content

## Dailies

### 2026-02-21-Sat

|<-- cursor here

... some content with out any lines starting with `### `
```

#### End-of-File

Beginning with:

```md
# My Project Title

... some content

## Dailies{eof}
```

The macro would result in:

```md
# My Project Title

... some content

## Dailies

### 2026-02-21-Sat

|<-- cursor here

{eof}
```

## Usage Instructions

### Prerequisites

1. Install the `geddski.macros` extension from the VS Code Marketplace
2. PowerShell is required (already included with Windows)

### Workspace Setup

For each workspace where you want to use these macros, you need to add two files to the `.vscode` folder:

#### File 1: `.vscode/insert-daily-entry.ps1`

Create this file with the PowerShell script that handles the date insertion. (See the file in this workspace for the complete code.)

#### File 2: `.vscode/tasks.json`

Create or update this file to include the "Insert Daily Entry" task. (See the file in this workspace for the complete configuration.)

### User Settings Configuration (One-Time Setup)

To make this macro available across all your VS Code instances:

#### Step 1: Add the Macro to User Settings

1. Press `Ctrl+Shift+P` to open the Command Palette
2. Type and select **"Preferences: Open User Settings (JSON)"**
  * `%APPDATA%\Code\User\settings.json`
3. Add the following `"macros"` section to your settings (merge with existing settings if present):

```json
"macros": {
    "insertDailyEntry": [
        {
            "command": "workbench.action.files.save"
        },
        {
            "command": "workbench.action.tasks.runTask",
            "args": "Insert Daily Entry"
        },
        {
            "command": "workbench.action.files.revert"
        }
    ],
    "goToLatestDaily": [
        {
            "command": "cursorTop"
        },
        {
            "command": "actions.find"
        }
    ]
}
```

4. Save the settings file

#### Step 2: Add the Keyboard Shortcut

1. Press `Ctrl+Shift+P` to open the Command Palette
2. Type and select **"Preferences: Open Keyboard Shortcuts (JSON)"**
  * `%APPDATA%\Code\User\keybindings.json`
3. Add the following keybindings (add to the array, separated by commas if other bindings exist):

```json
{
    "key": "ctrl+alt+d",
    "command": "macros.insertDailyEntry",
    "when": "editorTextFocus"
},
{
    "key": "ctrl+alt+c",
    "command": "macros.goToLatestDaily",
    "when": "editorTextFocus"
}
```

4. Save the keybindings file

#### Step 3: Reload VS Code

1. Press `Ctrl+Shift+P` to open the Command Palette
2. Type and select **"Developer: Reload Window"**
3. Wait for VS Code to reload

### Using the Macros

#### Insert Daily Entry (`Ctrl+Alt+D`)

1. Open any markdown file containing a `## Dailies` section
2. Press `Ctrl+Alt+D`
3. The macro will insert a new daily entry with today's date

#### Go to Latest Daily Entry (`Ctrl+Alt+C`)

This is a helper macro that positions your cursor at the top of the document and opens the find dialog, making it quick to search for the latest date entry:

1. Press `Ctrl+Alt+C`
2. Type `### 20` in the find box that appears
3. Press `Enter` to jump to the first (newest) match
4. Press `Esc` to close the find dialog
5. Position your cursor as needed

**Why this approach?**
Due to limitations in the `geddski.macros` extension (it cannot programmatically type into VS Code dialogs or execute JavaScript), full automation isn't possible. However, this semi-automated approach is still much faster than manually scrolling, especially in large files.

**Tips:**
- After pressing `Ctrl+Alt+D` to insert a new entry, press `Ctrl+Alt+C` and search for `### 20` to quickly jump to it
- VS Code remembers your last search, so subsequent uses will have `### 20` pre-filled

## Troubleshooting

### Python Extension Popup

If you see a popup asking to install the Python Environments extension (`ms-python.vscode-python-envs`) when running the macros, this is due to VS Code detecting the `.ps1` PowerShell script files. You can:

1. **Dismiss the popup** - Click the X to close it. The macros will still work correctly.
2. **Install the extension** - If you work with Python, you can install it, though it's not required for these macros.
3. **Disable extension recommendations** - Add the following to your workspace settings (`.vscode/settings.json`):
   ```json
   {
       "extensions.ignoreRecommendations": true
   }
   ```

The popup does not affect the functionality of the macros.

