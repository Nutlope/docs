# Settings

Fig settings are located in `~/.fig/settings.json`. Most updates will be reflected immediately. Some will require restarting the app. 

Note that `~/.fig/settings.json` must not contain comments.


## Updating Settings Using the CLI

You can read `fig` CLI, using the following syntax:
```bash
fig settings key
```

You can update setting values from the CLI as well:
```bash
fig settings key value
```

Note that `value` must be valid JSON. If a value cannot be parsed as a JSON literal, it will be interpreted as a `string`.
As some shells interpret JSON characters, it is often helpful to wrap the value in single quotes to ensure it isn't evaluated. 


## List of Setting Keys
All Fig settings are included below. 

### Autocomplete

**autocomplete.developerMode** `[bool]` 

> When `true`, Fig will not cache completion specs and will load specs from the directory specified in `autocomplete.devCompletionsFolder` rather than `~/.fig/autocomplete`.

**autocomplete.theme** `[light | dark]` 

> When `light`, Fig will run in light mode. When `dark`, Fig will run in dark mode.


**autocomplete.sortMethod** `[string]`

> `recency`: [Default] Sort by most recently used. <br></br> `alphabetical`: Sort by alphabetical

**autocomplete.devCompletionsFolder** `[directory]` 

> When `autocomplete.developerMode` is enabled, Fig loads completion specs from the specified directory.

**autocomplete.scrollWrapAround** `[bool]`

> A flag that determines whether the selection will wrap around when pressing arrow key at bottom or top of list.

**autocomplete.insertSpaceAutomatically** `[bool]`

> A flag that determines whether Fig will automatically insert a space.

**autocomplete.immediatelyRunDangerousCommands** `[bool]`

> A flag that determines whether Fig will present suggestions to immediately run commands that might be dangerous, like `rm`.

**autocomplete.immediatelyRunGitAliases** `[bool]`

> A flag that determines whether Fig will present suggestions to immediately run `git` aliases.

**autocomplete.disableForCommands** `[array]`

> Pass in a JSON array of commands Fig should not autocomplete on.

**autocomplete.enter** `[insert | ignore]`

> Set the behavior of the enter key.
> 1. `insert` - pressing enter will insert selected suggestion
> 2. `ignore` - pressing enter will run whatever command is currently in the terminal. (Fig will not intercept the keystroke.)

**autocomplete.tab** `[insertOrPrefix | insert | shake | navigate]`

> Set the behavior of the tab key.
> 1. `insert` - pressing tab will insert selected suggestion
> 2. `insertOrPrefix` - pressing tab will insert selected suggestion or common prefix of all suggestions, if it exists.
> 3. `shake` - pressing tab will insert common prefix, if it exists. Otherwise, it will indicate that there is no shared prefix by shaking.
> 4. `navigate` - pressing tab will insert common prefix, if it exists. Otherwise, it will select the next suggestion in the list.

**autocomplete.width** `[number]`

> Set the maximum width of the autocomplete window.

**autocomplete.height** `[number]`

> Set the maximum height of the autocomplete window.

### Pseudoterminal

**pty.path** `[string]` 

> The `$PATH` variable used in pseudoterminals. If autocomplete isn't showing file suggestions, running `fig settings pty.path "$PATH"` may fix the issue. *You will need to restart Fig for the changes to apply.*

**pty.rc** `[filepath]`

> A file that will be sourced when Fig creates a pseudoterminal

### Application

**app.launchOnStartup** `[bool]` 

> A flag that determins whether the Fig app is added to Login Items. If `true`, Fig will launch automatically whenever you restart your computer.

**app.hideMenubarIcon** `[bool]`

> Hide Fig's icon ◧ in the Mac status bar

**app.disableTelemetry** `[bool]` 

> Opt-out of all non-essential telemetry collection. By default, Fig collects limited usage information to provide support and improve the product. Read our [statement on privacy](https://withfig.com/privacy) for more details. 
**Note**: Fig will still send one ping a day with aggregate usage metrics (time spent in terminal, number of times the autocomplete window appeared, number of times autocomplete suggestions were inserted, number of total commands run) as well as crash reports.