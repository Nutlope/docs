# API

## Root Object

The root object is used to define every completion spec. It must be declared as a `var`, not a `const` or `let`. It must also be called `completionSpec`. `completionSpec` is actually a [Subcommand Object](#subcommand-object).

```ts
// Must be "var" NOT "let" or "const")
// Must be "completionSpec" 
export const completionSpec: Fig.Spec = { // this is a Subcommand Object
  name: "git",
  description: "the stupid content tracker"
	...
}
```

## Suggestion Object

Fig takes the user's input, maps it to your completion spec, then uses the completion spec to output an array of suggestion objects. Each item you see in the autocomplete menu below (with an icon, name, and description) is a suggestion object. You can customize a suggestion object to change its UI and the value that's inserted into the terminal when the user presses enter/tab.

![](/docAssets/autocomplete/api/suggestionObjects.png)

You will almost never make Suggestion Objects. Instead, you will make a completion spec of [Subcommand](#subcommand-object), [Option](#option-object), and [Arg Objects](#arg-object). Fig will then use these objects to generate Suggestion Objects.

<!-- START-ID: BaseSuggestion -->
| Property Name | Type | Required | Description |
|---|---|---|---
| displayName | string | ☐ |  The string that is displayed in the UI for a given suggestion. This Overrides the name property.    For the npm CLI we have a subcommand called `install`. If we wanted  to display some custom text like `Install an NPM package 📦` we would set  `name: "install"` and `displayName: "Install an NPM package 📦"` |  |
| insertValue | string | ☐ |  The value that's inserted into the terminal when a user presses enter/tab or clicks on a menu item.  You can optionally specify {cursor} in the string and Fig will automatically place the cursor there after insert.  The default is the name prop.    For `git commit` the `-m` option has an insert value of `-m '{cursor}'` |  |
| description | string | ☐ |  The text that gets rendered at the bottom of the autocomplete box.  Keep it short and simple! |  |
| icon | string | ☐ |  The icon that is rendered is based on the type, unless overwritten. Icon  can be a 1 character string, a URL, or Fig's icon protocol (fig://) which lets you generate  colorful and fun systems icons: https://withfig.com/docs/autocomplete/reference/icon-api    `A`, `😊`  `https://www.herokucdn.com/favicon.ico`  `fig://icon?type=file` |  |
| isDangerous | boolean | ☐ |  Specifies whether the suggestion is "dangerous". If so, Fig will not enable  its "insert and run" functionality (when Fig has the red insert icon).  This will make it harder for a user to accidentally run a dangerous command.    This is used in specs like rm and trash. |  |
| priority | number | ☐ |  The priority for a given suggestion determines its ranking in the Fig popup. A higher ranked priority will be listed first. The min priority is 0. The max priority is 100. The default priority is 50.  If a given suggestion has a priority between 50 and 75 (inluding the default 50) AND has been selected by the user before, the prioritiy will be replaced with 75 + the timestamp of when that suggestion was selected as a decimal.  If a given suggestion has a priority outside of 50-75 AND has been selected by the user before, the prioritiy will be increased by the timestamp of when that suggestion was selected as a decimal.     If you want your suggestions to always be at the top order regardless of whether they have been selected before or not, rank them 76 or above  If you want your suggestions to always be at the bottom regardless of whether they have been selected before or not, rank them 49 or below |  |
| hidden | boolean | ☐ |  Specifies whether a suggestion should be hidden from results. Fig will only show it if the user types the exact same thing as the name    The "-" suggestion is hidden in the `cd` spec. You will only see it if you type cd - |  |
<!-- END-ID: BaseSuggestion -->

## Subcommand Object

Used to define subcommands under a main command. For example, within `git commit`, `commit` is a subcommand of `git`.

Subcommand Objects are recursive by nature. 

<!-- START-ID: Subcommand -->
| Property Name | Type | Required | Description |
|---|---|---|---
| name | string | ☑ |  The exact name of the subcommand. It is important to get this right for parsing purposes.    For npm, the subcommand is `npm install` would have "name: install" (no extra spaces or characters, exactly like this) |  |
| subcommands | Subcommand[] | ☐ |  A list of subcommands for this spec.  Subcommands can be nested within subcommands. |  |
| options | Option[] | ☐ |  A list of option objects for this subcommand. |  |
| args | SingleOrArray<Arg> | ☐ |  An array of args or a single arg.   **Important**  If a subcommand takes an argument, please at least include an empty Arg Object  (e.g. `{}`). If you don't, Fig will assume the subcommand does not take an argument and we will present the wrong suggestions.    git push takes two arguments. The most basic representation of this is  `args: [{}, {}]` |  |
| additionalSuggestions | Suggestion[]  | ☐ |  A list of Suggestion objects that are appended to a specific subcommand. Often these are shortcuts    `commit -m '{cursor}'` is a shortcut for git |  |
| loadSpec | string | ☐ |  Allows Fig to refer to another completion spec in the `~/.fig/autocomplete` folder.  Specify the spec name without `js`. This is simiar but different to isCommand in the Arg object so read both carefully    `aws-s3` refer to the `~/.fig/autocomplete/aws-s3` spec.   When is this used? The aws spec is so large that it is slow to load. It needs to be  brokenup into a separate spec for each subcommand.   If your CLI tool takes another CLI command (e.g. time , builtin... ) or a script  (e.g. python, node) and you would like Fig to continue to provide completions for this  script, see `isCommand` and `isScript` in {@link {https://withfig.com/docs/autocomplete/api#arg-object | Arg}. |  |
| generateSpec | Function<string[], Promise<Spec>> | ☐ |  Dynamically generate a completion spec to be merged in at the same level as the current subcommand. This is useful when a CLI is generated dynamically. This is a function that takes in an array of strings (the tokens the user has typed) and outputs a completion spec object    Laravel artisan has its own subcommands but also lets you define your own completion spec. |  |
<!-- END-ID: Subcommand -->




## Option Object

Options add additional information to a subcommand. They usually start with `-` or `--`. 

* Some options are boolean: they are there or not. These are called flags. e.g. `git --verion` . 
* Some options take an argument e.g. `git commit -m <message>`

<!-- START-ID: Option -->
| Property Name | Type | Required | Description |
|---|---|---|---
| name | SingleOrArray<String> | ☑ |  The exact name(s) of the option. It is very important to get this right for parsing purposes. It can be a string or an array of strings.  **Important**: do NOT include an = sign here. Fig handles this logic for you. Including one will mess up the parsing.    For git commit -m, the option name is `["-m", "--message"]` |  |
| args | SingleOrArray<Arg> | ☐ |  An array of args or a single arg object.   **Important**  If an option takes an argument, please at least include an empty Arg Object  (e.g. `{}`). If you don't, Fig will assume the option does not take an argument and will present the wrong suggestions.    `git commit -m` takes one argument. The most basic representation of this is  `args: {}` |  |
<!-- END-ID: Option -->


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


<!-- START-ID: Arg -->
| Property Name | Type | Required | Description |
|---|---|---|---
| name | string | ☐ |  The name of an argument. This is different to the `name` prop for subcommands, options, and suggestion objects so please read carefully.  This is a normal, human readable string that signals to the user the type of argument they are inserting if there are no available suggestions.  Fig does not use this value for parsing. Therefore, it can be whatever you want.    The name prop for the `git commit -m` arg object is "message". You can see this when you type! |  |
| description | string | ☐ |  The text that gets rendered at the bottom of the autocomplete box a) when the user is inputting an argument and there are no suggestions and b) for all generated suggestions for an argument  Keep it short and direct! |  |
| isDangerous | boolean | ☐ |  Specifies whether the suggestions generated by this argument are "dangerous". If so, Fig will not enable  its insert and run functionality (when the red icon appears that automatically executes commands for you rather than just inserting)    This is used in specs like rm and trash. |  |
| suggestions | string[]  | ☐ |  An array of strings or Suggestion obejcts. Use this prop to specify custom suggestions  that are static (ie you know of in advance and don't have to be statically generated).  If suggestions are dependent upon the user's input or context, you most likely will  want to use a Generator object in this arg's generator prop. |  |
| template | Template | ☐ |  Fig has pre-built generators for common suggestion types. Currently, we support  templates for either "filepaths" or "folders". You can do either of these as a string or both in an array.  Folders will only show folders. Filepaths will show folders and filepaths but will only offer the insert and execute functionality (the red automatic insert icon you see when using cd) for files NOT folders.    `cd` uses the `folders` template whereas ls uses `[filepaths, folders]` |  |
| generators | SingleOrArray<Generator> | ☐ |   Generators let you run shell commands on the user's device to generate suggestions for arguments.  The generators prop takes a single generator object or a list of generator objects.  The generator object outputs an array of suggestions which are offered to users when inserting an argument |  |
| variadic | boolean | ☐ |  Specifies that the argument is variadic and therefore repeats infinitely.    `echo` takes a variadic argument (`echo hello world ...`) so does `git add` |  |
| isOptional | boolean | ☐ |  True if an argument is optional. It is important you include this for our parsing. If you don't, Fig will assume the argument is mandatory and will not offer suggestions for a user.    Git push [remote] [branch] takes two optional args |  |
| isCommand | boolean | ☐ |  Specifies that the argument is an entirely new command which Fig should start completing on from scratch    `time` and `builtin` have only one argument and this argument has the `isCommand` property. If I type `time git`, Fig will load up the git completion spec because the isCommand property is set. |  |
| isScript | boolean | ☐ |  Exactly the same as the `isCommand` prop except Fig will look for a completion spec in a .fig folder in the user's current working directory.    `python` take one argument which is a `.py` file. If I have a `main.py` file on my desktop and my current working directory is my desktop, if I type `python main.py` Fig will look for a completion spec in `~/Desktop/.fig/main.py.js`  See our docs for more on this {@link https://withfig.com/docs/autocomplete/autocomplete-for-teams | Fig for Teams} |  |
| debounce | boolean | ☐ |  This will debounce every keystroke event for this particular arg. If there are no keystroke events after 100ms, Fig will execute all the generators in this arg and return the suggestions.    NPM install and pip install send debounced network requests after inactive typing from users. |  |
<!-- END-ID: Arg -->

**Note**: To re-iterate again, an Arg Object can be completely empty (e.g. `{}`). So if a subcommand or option takes an argument, please include it otherwise Fig's parsing will break!



## Generator Object

Generators are used to programatically generate suggestion objects. For example, they can be used to fetch and suggest a list of git remotes, grab all the folders in the current working directory, or even hit an API endpoint.

For more details on generators and their properties, see [Generators](/docs/autocomplete/generators).

<!-- START-ID: Generator -->
| Property Name | Type | Required | Description |
|---|---|---|---
| template | Template | ☐ |  Fig has pre-built generators for common suggestion types. Currently, we support  templates for either "filepaths" or "folders". You can do either of these as a string or both in an array.  Folders will only show folders. Filepaths will show folders and filepaths but will only offer the insert and execute functionality (the red automatic insert icon you see when using cd) for files NOT folders.    `cd` uses the `folders` template whereas ls uses `[filepaths, folders]` |  |
| filterTemplateSuggestions | Function<Suggestion[], Suggestion[]> | ☐ |  This function takes a single argument: the array of suggestion objects output by the template prop. It then lets you edit them as you see fit. You must then return an array of suggestion objects.   The python spec has an arg object which has a template for "filepaths" and then filters out all suggestions generated that don't end with "/" (to keep folders) or ".py" (to keep python files) |  |
| script | StringOrFunction<string[], string> | ☐ |  In order to generate contextual suggestions for arguments, Fig lets you execute a shell command on the users local device as if it were done in their current working directory.  You can either specify  1. a string to be executed (like `ls` or `git branch`)  2. a function to generate the string to be executed. The function takes in an array of tokens of the user input and should output a string. You use a function when the script you run is dependent upon one of the tokens the user has already input (for instance an app name, a kubernetes context etc)  After executing the script, the output will be passed to one of `splitOn` or `postProcess` for further processing to produce suggestion objects.    `git checkout` takes one argument which is a git branch. Its arg object has a generator with a script of `git branch`  `heroku features:disable --app ABC FEATURE` the suggestion for the FEATURE argument are dependent upon the --app given (in this case ABC). Therefore, we would use a function to generate the script we would run here |  |
| splitOn | string | ☐ |  A synctactic sugar over postProcess. This takes in the text output of `script`, splits it on the string you provide here, and the automatically generates an array of suggestion objects for each item.    Specify "," or "\n", and Fig will do the work of the `postProcess` prop for you |  |
| postProcess | Function<string, Suggestion[]> | ☐ |  This function takes one paramater: the output of `script`. You can do whatever processing you want, but you must return an array of Suggestion objects. |  |
| trigger | string  | ☐ |  Fig performs numerous optimizations to avoid running expensive shell functions many times. For instance, after you type `cd[space]` we load up a list of folders (the suggestions). After you start typing, we instead filter over this list of folders (the filteredSuggestions).  The suggestions remain the same while the filteredSuggestions change on each input.   Typically, Fig regenerates the suggestions every time the user hits space as in bash, a space typically delimits commands. However, if the `trigger` prop is defined, Fig will run the trigger function on each keystroke. If it returns true, instead of filtering over the suggestions, Fig will regenerate the list of suggestions THEN filter over them.  The trigger function takes two inputs: the new token the user typed and the token on the keystroke before.   The trigger prop can also be a simple string. This is synctactic sugar that allows you to specify a single character. If count of this character in the string before !== the count of the new string, Fig will regenerate the suggestions.   Using a trigger is especially beneficial when you have an argument contained inside a single string that is not separated by a space. It is often used with a custom prop or script (as a function)  Finally, make sure you don't confuse trigger with debounce. Debounce will regenerate suggestions after a period of inactivity typing. Trigger will regenerate suggestions when the function you define returns true!   Use some logging in the function to work out when trigger is being run    You can see the trigger in action every time you use file and folder completions (e.g. with `cd`). When you type a `/`, Fig will regenerate its list of file and folder suggestions by appending the path of what you've already typed to your current working directory.  e.g. If I had already typed "desktop". The current list of suggestions is from the ~ directory and the filterTerm is "desktop". Then I type "/" so it says "desktop/", the trigger would return true, Fig will generate suggestions for the directory `~/desktop/` and the filterTerm will become an empty string.  |  |
| filterTerm | StringOrFunction<string, string> | ☐ |  Read the note above about how triggers work. Triggers and filterTerm may seem similar but are actually different. The trigger defines when to regenerate new suggestions. The filterTerm defines what characters we should use to filter over these suggestions.   It can be a function: this takes in what the user has currently typed as a string and outputs a separate string that is used for filtering  It can also be a string: this is synctactic sugar that takes everything in the string after the character(s) you choose.   Use some logging in the function to work out what the trigger is    cd has a filter term of "/". If an argument to `cd` includes a "/" Fig will filter over all of the suggestions generated using the string AFTER the last "/"  |  |
| custom | Function<string[], Promise<Suggestion[]>> | ☐ |  Custom function is a bit like script as a function, however, it gives you full control.   It is an async function.   It takes on argument: an array of tokens of what the user has typed  It must return an array of suggestion obejcts.   The function also allows the use of the async function `executeShellCommand` which executes commands from the same current working directory as the user and returns the output as a text blob.    ```  custom: (context) => {     var out = await executeShellCommand("ls")     return out.split("\n").map((elm) => {name: elm})  }  ``` |  |
| cache | Cache | ☐ |  For commands that take a long time to run, Fig gives you the option to cache their response. You can cache the response globally or just by the directory they were run in  You just need to specify a `ttl` (time to live) for how long the cache will last (this is a number)  You can also optionally turn on the ability to just cache by directory (`cacheByDirectory: true`)    The kubernetes spec makes use of this.  |  |
<!-- END-ID: Generator -->

**Note**: At least one of `template`, `script`, or `custom` is **required**. Only one of these three props is allowed per generator.

For more details on generators and their properties, see [Generators](/docs/autocomplete/generators).

