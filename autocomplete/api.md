# API

## Root Object

The root object is used to define every completion spec. It must be declared as a `var`, not a `const` or `let`. It must also be called `completionSpec`. `completionSpec` is actually a [Subcommand Object](#subcommand-object).

```js
// Must be "var" NOT "let" or "const")
// Must be "completionSpec" 
var completionSpec = { // this is a Subcommand Object
  name: "git",
  description: "the stupid content tracker"
	...
}
```

## Suggestion Object

Fig takes the user's input, maps it to your completion spec, then uses the completion spec to output an array of suggestion objects. Each item you see in the autocomplete menu below (with an icon, name, and description) is a suggestion object. You can customize a suggestion object to change its UI and the value that's inserted into the terminal when the user presses enter/tab.

![](/docAssets/autocomplete/api/suggestionObjects.png)

You will almost never make Suggestion Objects. Instead, you will make a completion spec of [Subcommand](#subcommand-object), [Option](#option-object), and [Arg Objects](#arg-object). Fig will then use these objects to generate Suggestion Objects.

| Property Name | Type                       | Required | Default                                                      | Description                                                  |
| ------------- | -------------------------- | -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| name          | string or array of strings | ☑        |                                                              | The text that’s rendered in each row of the dropdown.<br /><br />**A quick note on the `name` prop**<br />Fig uses the `name` prop for parsing purposes. It is important the `name` prop exactly matches the CLI tool. If you want to customise it what is says in the dropdown, please use `displayName` |
| displayName   | string                     | ☐        | the name prop                                                | Overrides the `name` property. <br />e.g. for the `npm` CLI we have a subcommand called `install`. If we wanted to display custom text like `Install an NPM package 📦` we would set `name: "install"` & `displayName: "Install an NPM package 📦"` |
| insertValue   | string                     | ☐        |                                                              | The value that's inserted into the terminal when a user presses enter/tab or clicks on a menu item <br /><br />You can optionally specify `{cursor}` in the string and Fig will automatically place the cursor there after insert.<br />e.g. for `git commit` the  `-m`  option has an insert value of `-m '{cusor}' `<br />If the `name` prop is set, Fig will automatically |
| description   | string                     | ☐        | null                                                         | The text that gets rendered at the bottom of the autocomplete box. Keep it short and direct! |
| type          | string                     | ☐        | null                                                         | The type of suggestion, one of folder, file, arg, subcommand, option, special. |
| icon          | string                     | ☐        | auto-generated based on type prop. If type doesn't exist, will be `?` | The icon that is rendered is based on the type, unless overwritten. Icon can be a 1 character string (e.g. `A` or `😊`) , a URL, or [Fig's icon protocol](/docs/autocomplete/icon-api) (`fig://`) which will get mac system icons |
| isDangerous   | bool                       | ☐        | false                                                        | Specifies whether the suggestion is "dangerous". If so, Fig will not enable its insert and run functionality whereby selecting a suggestion runs a command. This will make it harder for a user to accidentally run a dangerous command. This is used in specs like `rm` and `trash` |



## Subcommand Object

Used to define subcommands under a main command. For example, within `git commit`, `commit` is a subcommand of `git`.

Subcommand Objects are recursive by nature. 

| Property Name         | Type                               | Required | Default             | Description                                                  |
| --------------------- | ---------------------------------- | -------- | ------------------- | ------------------------------------------------------------ |
| name                  | string or array of strings         | ☑        |                     | Refer to `name` under [Suggestion Object](#suggestion-object) |
| displayName           | string                             | ☐        | the name prop       | Refer to `displayName` under [Suggestion Object](#suggestion-object) |
| insertValue           | string                             | ☐        | the name prop       | Refer to `insertValue` under [Suggestion Object](#suggestion-object) |
| description           | string                             | ☐        | null                | Refer to `description` under [Suggestion Object](#suggestion-object) |
| subcommands           | array of Subcommand objects        | ☐        |                     | Refer to [Subcommand Object](#subcommand-object) for the subcommand schema |
| icon                  | string                             | ☐        | the subcommand icon | Refer to `icon` under [Suggestion Object](#suggestion-object) |
| isDangerous           | bool                               | ☐        | false               | Refer to `isDangerous` under [Suggestion Object](#suggestion-object) |
| options               | array of Option objects            | ☐        |                     | Refer to [Option Object](#option-object) for the Option schema |
| args                  | Arg object or array of Arg objects | ☐        |                     | Refer to [Arg Object](#arg-object) for the Arg schema. <br /><br />**Important**: If a subcommand takes an argument, please at least include an empty Arg Object (e.g. `{}`). If you don't, Fig will assume the subcommand does not take an argument. This means Fig will present the wrong suggestions. |
| additionalSuggestions | array of Suggestion objects        | ☐        |                     | Refer to [Suggestion Object](#suggestion-object) for the suggestion schema. Use additionalSuggestions to make custom suggestions. Note, you should only use this for special cases. Most likely, what you are trying to accomplish should be done with the `args` prop |
| loadSpec              | string                             | ☐        |                     | Allows Fig to refer to another completion spec in the `~/.fig/autocomplete` folder. Specify the spec name without js. e.g. `aws-s3` refer to the `~/.fig/autocomplete/aws-s3` spec. <br /><br />When is this used? The `aws` spec is so large that it is slow to load. It needs to be broken up into a separate spec for each subcommand.<br /><br />If your CLI tool takes another CLI command (e.g. `time` , `builtin`... ) or a script (e.g. `python`, `node`) and you would like Fig to continue to provide completions for this script, see `isCommand` and `isScript` in [Arg Object](#arg-object) |





## Option Object

Options add additional information to a subcommand. They usually start with `-` or `--`. 

* Some options are boolean: they are there or not. These are called flags. e.g. `git --verion` . 
* Some options take an argument e.g. `git commit -m <message>`

| Property Name         | Type                               | Required | Default | Description                                                  |
| --------------------- | ---------------------------------- | -------- | ------- | ------------------------------------------------------------ |
| name                  | string or array of strings         | ☑        |         | The same as  `name` under [Suggestion Object](#suggestion-object) however, you may also include an array strings e.g. `["-m", "--message" ]` |
| displayName           | string                             | ☐        |         | Refer to `displayName` under [Suggestion Object](#suggestion-object) |
| insertValue           | string                             | ☐        |         | Refer to `insertValue` under [Suggestion Object](#suggestion-object) |
| description           | string                             | ☐        |         | Refer to `description` under [Suggestion Object](#suggestion-object) |
| icon                  | string                             | ☐        |         | Refer to `icon` under [Suggestion Object](#suggestion-object) |
| isDangerous           | bool                               | ☐        | false   | Refer to `isDangerous` under [Suggestion Object](#suggestion-object) |
| args                  | Arg object or array of Arg objects | ☐        |         | Refer to [Arg Object](#arg-object) for the Arg schema. <br /><br />**Important**: If an option takes an argument, please at least include an empty Arg Object (e.g. `{}`). If you don't, Fig will assume the option does not take an argument. This means Fig will present the wrong suggestions. |
| additionalSuggestions | array of Suggestion objects        | ☐        |         | Refer to [Suggestion Object](#suggestion-object) for the suggestion schema. Use additionalSuggestions to make custom suggestions. In most cases, what you are trying to accomplish should be done with the `args` prop |



## Arg Object

The Arg Object is a blueprint used to generate dynamic suggestions for users based on their context. e.g. when a user types `git push [remote] [branch]` they expect to first see suggestions for all the remote repos connected to their local git repo, then all of the local branches connected to their local git repo. 

**Important:** Fig relies on Subcommand and Option objects correctly including an Arg Objects when relevant. This is important for our parsing and generating suggestions. e.g. in `git commit -m 'my message'` we see that `-m` is an option that takes an argument. If we don't specify that `-m` takes an arg, when the user starts typing their commit message, Fig will begin generating suggestions for the wrong thing. 

Ideally your arg object includes a name, description, and generator, however, if you are just building the skeleton of a CLI tool, It is okay to specify an empty Arg Object e.g.

```javascript
// dummy option object
{
	name: "-m"
	description: "commit message"
	args: {} // empty Arg Object
}
```



| Property Name | Type                                            | Required | Default                                                      | Description                                                  |
| ------------- | ----------------------------------------------- | -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| name          | string                                          | ☐        |                                                              | Refer to `name` under [Suggestion Object](#suggestion-object) |
| displayName   | string                                          | ☐        | the name prop                                                | Refer to `displayName` under [Suggestion Object](#suggestion-object) |
| insertValue   | string                                          | ☐        | the name prop                                                | Refer to `insertValue` under [Suggestion Object](#suggestion-object) |
| description   | string                                          | ☐        | null                                                         | Refer to `description` under [Suggestion Object](#suggestion-object) |
| suggestions   | array of strings or array of Suggestion objects | ☐        | null                                                         | Refer to [Suggestion Object](#suggestion-object) for the suggestion schema. Use this prop to specify custom suggestions that aren't dependent upon the user's input or context. You most likely will want to use a generator Object in the `generator` to create suggestions dynamically. |
| icon          | string                                          | ☐        | auto-generated based on type prop. If type doesn't exist, will be `?` | Refer to `icon` under [Suggestion Object](#suggestion-object) |
| isDangerous   | bool                                            | ☐        | false                                                        | Refer to `isDangerous` under [Suggestion Object](#suggestion-object) |
| isOptional    | bool                                            | ☐        | false                                                        | True if a CLI command is optional                            |
| template      | string                                          | ☐        |                                                              | Fig has pre-built `generators` for common suggestion types. Currently, we support templates for either `"filepaths"` or `"folders"`. e.g. `cd` uses a template for `folders` |
| generators    | Generator object or array of Generator objects  | ☐        |                                                              | Generators let you run shell commands on the user's device to *generate s*uggestions for arguments |
| variadic      | boolean                                         | ☐        | false                                                        | Specifies that the argument is variadic and therefore repeats infinitely. e.g. `echo` takes a variadic argument (`echo hello world ...`) |
| isCommand     | boolean                                         | ☐        | false                                                        | Specifies that the argument is an entirely new command which Fig should complete on. e.g. `time` and `builtin` have only one argument and this argument has the isCommand property |
| isScript      | boolean                                         | ☐        | false                                                        | Specifies that the argument is a script which Fig may complete on. e.g. `python`  takes one argument which is a `.py` file. It is possible for Fig to offer for completions on this .py file. See [Fig for Teams](/docs/autocomplete/autocomplete-for-teams) |

**Note**: To re-iterate again, an Arg Object can be completely empty (e.g. `{}`). So if a subcommand or option takes an argument, please include it otherwise Fig's parsing will break!



## Generator Object

Generators are used to programatically generate suggestion objects. For example, they can be used to fetch and suggest a list of git remotes, grab all the folders in the current working directory, or even hit an API endpoint.

For more details on generators and their properties, see [Generators](/docs/autocomplete/generators).

| Property Name             | Type                                                         | Required | Description                                                  |
| ------------------------- | ------------------------------------------------------------ | -------- | ------------------------------------------------------------ |
| template                  | string                                                       | ☐        | Must be either "filepaths" or "folders"                      |
| filterTemplateSuggestions | function <br />Input: array of Suggestion objects <br />Output: array of Suggestion objects | ☐        | Function that lets you filter the Suggestion objects output by the template. |
| script                    | string OR function<br /><u>function</u> <br />Input: array of strings <br />Output: string | ☐        | The shell command to execute in the user's current working directory. The output is a string. It is then converted into an array of Suggestion objects using `splitOn` or `postProcess` |
| splitOn                   | string                                                       | ☐        | As splitting the output of `script` is such a common use case for `postProcess`, we build the `splitOn` property. Simply define a string to split the output of script on (e.g. `","` or `"\n"` and Fig will do the work of the `postProcess` prop for you. |
| postProcess               | function<br />Input: string<br />Output: array of Suggestion objects | ☐        | Define a function that takes a single input: the output of executing script. This function then return an array of Suggestion objects that will be rendered by Fig. |
| custom                    | async function <br />Input: array of strings<br />Output: array of Suggestion objects | ☐        | Custom Functions let you define a function that takes an array of the user's input, run multiple shell commands on the user's machine, and then generate suggestions to display. |
| trigger                   | string or function<br /><u>function</u><br />Inputs: string, string<br /> Output: boolean | ☐        | Defines a trigger that determines when to regenerate suggestions for this argument by re-running the generators. |
| filterTerm                | string or function <br /><u>function</u> <br />Input: string <br />Output: string | ☐        | A function to determine what part of the string the user is currently typing we should use to filter our suggestions. |

**Note**: At least one of `template`, `script`, or `custom` is **required**. Only one of these three props is allowed per generator.

For more details on generators and their properties, see [Generators](/docs/autocomplete/generators).

