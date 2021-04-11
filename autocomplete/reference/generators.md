# Generators

## Quick Summary

Generators let you run shell commands on the user's device to *generate* suggestions for arguments

- There are 4 types of generators (see below)
- Fig runs all shell commands in a separate login-shell at the current working directory of the user's terminal
  - [Executing Shell Commands](./executing-shell-commands)
- Generators are defined in the `generators` property of an Argument object.
  - The `generators` property can be a single Generator object or an array of Generator objects.

### Examples

- `cd [folder]`
  - `cd` takes one argument: a single folder
    - Arg object 1: A generator that runs  `ls` to get the list of folders at the user's current directory
- `git push [remote] [branch]`
  - `git push` takes two arguments: a remote repo and a branch
    - Arg object 1: Include a generator that runs `git remote` as a script
    - Arg object 2: Include generator that runs `git branch` as a script

## Types of Generators

There are 4 types of generators. At the end of the day, they all give you the ability to define what shell command(s) to run. Then given that output, parse it however you'd like to produce Suggestion objects.

They types of Generator are listed below in order of customisability.

We would say the vast majority of Generators you make will be using **Templates** and/or **Script as a String**

### Templates

Filepaths or folders are very common arguments for subcommands. e.g.

- `git add [file or folder path]`
- `cd [folder path]`
- `open [file or folder path]`
- `ls -l [file or folder path]`

Because it's so common, Fig has predefined these arguments for you as a template.

```js
// Define templates inside generators in order
// to use the filterTemplateSuggestions function

args: {
  generator: {
    template: "filepaths",
    filterTemplateSuggestions: (suggestions) => { //optional
      var jsOnly = suggestions.filter((elm) => return elm.endsWith(".js"))
      return jsOnly
    }
  }
}

**OR**

// Define templates directly underneath args if you don't need to filter
// This also looks nicer and is easier to read
args: {
  template: "filepaths",
}
```

**Properties**

| Property                  | Data Type                                                    | Required | Description                                                  |
| ------------------------- | ------------------------------------------------------------ | -------- | ------------------------------------------------------------ |
| template                  | string                                                       | ☐        | One of: <br />- "filepaths": shows all files AND folders at current working directory (like running ls). <br />- "folders": only shows folders in current working directory (like running ls then filtering on.<br /><br />These templates also allow users to type / at the end of a directory and Fig will run show the files/folders in the new subdirectory. |
| filterTemplateSuggestions | function(suggestions: SuggestionObject[ ]): SuggestionObject | ☐        | A function that let's you filter the array of Suggestion objects output by the template. e.g. if you only wanted to show files ending in `.js` you could use a filter method inside.<br /><br />**Function Parameters**<br />suggestions: an array of Suggestion objects that was output by the template <br /><br />**Output**<br />an array of Suggestion objects that was output by the template |

**Notes & Tips**

- the `filterTemplateSuggestions` is only available if you define the the `template` inside a generator object
- if you define `template` outside a generator object, this will override
- If you don't plan on using `filterTemplateSuggestions`, we recommend defining `template` outside the generator object. It is easier to read inside the spec.
- Both templates have a trigger attached to them which cause Fig to re-run the suggestion object when a user types a `"/"`
  - e.g. if a user is in their Home (~) directory and inserts `cd Desktop/` Fig will render suggestions from the Desktop directory

### Script as a String

If you're suggesting something other than files or folders, we recommend using this. This let's you run a mini-script / command on the user's computer. The script runs in the user's current working directory and in the same shell. It's as if the user typed the command themselves.

The output from the script you specify is then converted to Suggestion objects. We can do this for you with `splitOn` or you can process the output yourself with `postProcess`. Not that you must specify at least one of `splitOn` or `postProcess`.

```js
args: {
  script: "git branch"
  splitOn: "\\n",
  // **OR**
  postProcess: (scriptOutput: string) => {
    var arr = scriptOutput.split("\\n")  
	  // return an array of Suggestion Objects
    return arr.map((elm) => {
      return {
        name: elm,
        icon: "🌱"
      }
    })
  },
    // [optional]
  trigger: (after: string, before: string) {
    if before.countSlash !== after.countSlash return true
    else return false
  }
}
```

**Properties**

| Property    | Data Type                                                    | Required | Description                                                  |
| ----------- | ------------------------------------------------------------ | -------- | ------------------------------------------------------------ |
| script      | string                                                       | ☐        | The shell command to execute in the user's current working directory. The output is a string. It is then converted into an array of Suggestion objects using `splitOn` or `postProcess`<br /><br /> e.g. git branch → show all the remote git repos available |
| postProcess | function(scriptOutput: string): SuggestionObject { } <br /><br />**Input**: output of executing script <br />**Output**: array of Suggestion objects | ☐        | **[Must specify one of postProcess or splitOn]** <br /><br />Define a function that takes a single input: the output of executing script. This function then return an array of Suggestion objects that will be rendered by Fig.  <br /><br />**Examples** <br />If `script` outputs data separated cleanly by newlines, or commas... <br />- split the input on new lines <br />- map or filter over your new array <br />- if each line had columns, split it again during the map<br />If `script` outputs json data,  use JSON.parse.<br /><br />**Other ideas** - use try/catch and return an empty array if there's an error |
| splitOn     | string                                                       | ☐        | As splitting the output of `script` is such a common use case for `postProcess`, we build the `splitOn` property. Simply define a string to split the output of script on (e.g. `","` or `"\n"` and Fig will do the work of the `postProcess` prop for you.  <br /><br />This is a simple and fast method to generate suggestions. If you want any more control over the parsing (like filtering) or you want more control over the UI, use `postProcess`. <br /><br />e.g. if the script was `git branch`, specifying splitOn = \n would split the output on new lines and automatically create a Suggestion object for each element. |
| trigger     | function                                                     | ☐        | See [Trigger](#Trigger)                       |
| filterTerm  | function or string                                           | ☐        | See [Filter Term](#filter-term)               |

**Notes & Hints**

- In `script`: You can chain multiple commands together in the one string and then use javascript to combine  their output. e.g. `echo "The quick"; echo "brown fox";`
- If a command goes to a pager (ie it scrolls instead of printing out normally) you can try piping it into `cat` e.g. `man git | cat`

### Script as a Function

This is *exactly* the same as **Script as a String**, except you can define what script to run as a **Function**.

Using Script as a Function may be necessary when you want to run scripts dynamically based on parts of the command that the user has entered. For example, Heroku requires 

The function takes a parameter, `context`, which includes an array of strings broken down into components. The strings can then be used as values to pass to the script you'd like to return.

When running `git checkout master`, `context` equals ["git", "checkout", "master"].

Context can discern different components including those grouped by quotes. When running `touch "new file"`, `context` equals ["touch", "new file"].

```js
args: {
  // Take an array of what the user has already input
  // e.g. 
  script: function (context: Array<string>) { 		
    var myScript; 
				
    if (contextArray) {
      // Return string
      return myScript
    },

    splitOn: "",
    // **OR**
    postProcess: (scriptOutput: string) => {			
  },
		
  // [optional]
  trigger: (after: string, before: string) {
  }
}
```

The properties below are exactly the same as **Script as a String** except for the `script` prop. In that case

**Properties**

| Property    | Data Type                                                    | Required | Description                                                  |
| ----------- | ------------------------------------------------------------ | -------- | ------------------------------------------------------------ |
| script      | function(context: array): string { }<br /><br /> **Input**: output of executing script <br />**Output**: array of Suggestion objects | ☐        | The shell command to execute in the user's current working directory. The output is a string. It is then converted into an array of Suggestion objects using `splitOn` or `postProcess`<br /><br /> e.g. git branch → show all the remote git repos available |
| postProcess |                                                              | ☐        | See [postProcess above](#script-as-a-string)                                    |
| splitOn     |                                                              | ☐        | See [splitOn above](#script-as-a-string)                                        |
| trigger     | function or string                                           | ☐        | See [Trigger](#trigger)                       |
| filterTerm  | function or string                                           | ☐        | See [Filter Term](#filter-term)                           |

### Custom Function

**Note:** The [Script as a String](#script-as-a-string) or [Script as a Function](#script-as-a-function) generator functions should suit your needs most of the time. Before writing a Custom Function, we suggest writing a simple version of the spec using generators first. If it captures > 60% or 70% of the use cases, worst case, Fig just won't show for the remaining use cases.

If you are going to write a Custom Function, please [email us](mailto:hello@withfig.com), we'd love to help.

--- 

Fig's completion spec standard accounts for the vast majority of CLI tools, but some features don't work with the standard spec. Custom Functions inside generators enable you to write more complex logic and generate suggestions based on user input unique to a certain CLI tool.

Custom Functions let you define a function that takes an array of the user's input, run multiple shell commands on the user's machine, and then generate suggestions to display. When using Custom Functions, you will likely have to work pretty closely with [Triggers](#trigger).

#### When to use Custom Functions

**1. An argument can be one of many types, and you want to provide suggestions once you have more context on what type the user is inputting**. 

e.g. `git checkout` supports multiple types of arguments

- `git checkout staging` → branch
- `git checkout h8ne3x` → commit hash
- `git checkout HEAD~` → Relative refspec

(If you're interested [here are a lot more](https://stackoverflow.com/a/18605496/2218728))

You could just use a **script as a string** Generator and only suggest branches by using `git branch`. This would be nice and would work fine. But it would be cool if you could type `HEAD^^` and have Fig tell you which commit description you are referring to. Custom functions let you do this.

**2. An argument takes its own sub-arguments that are not delimited by a space** 

e.g. in `npm` you can refer to a package in many ways AND then you can provide sub-arguments like the version, tag, and range, all within the same string.

![](../assets/autocomplete/generators/subargs.png)

Your function could do multiple regex matches on the argument the user is inputting. If it begins with an `@` symbol, then run a script to suggest a list of scopes. When the user types `/`, suggest a list of packages. As soon as the user types another `@` symbol, suggest a list of tags or versions associate with the input package name.

### Syntax

```js
args: {  
  generator: {
    custom: async function generateSuggestions(context: Array<string>): SuggestionObject {
      var currentArg = context[-1]
      var out;
      if (currentArg.includes(":")) {
        out = await runFigCommand("foo")
      }
      else {
        out = await runFigCommand("bar")
      }	
      return out.split("\\n") 
    },
    trigger: SEE BELOW,
    filterTerm: SEE BELOW
  }	
}
```

## Trigger

Defines a trigger that determines when to regenerate suggestions for this argument by re-running the generators.

By default, Fig runs the generators for an argument once when the user is about to enter the argument. Afterwards, as the user continues typing, suggestions are filtered from the original list of generated suggestions.

Trigger is used when we need the generators to regenerate all the unfiltered, base level suggestions.

**Types**

- String
  - e.g. if Trigger: "/", re-run the generators every time the last index of "/" is different between the previous input and the current input
- Function
  - Write a function to define custom logic for when a Trigger should fire.

```bash
fn(currentString: str, previousString: str): bool { }

e.g. fn("desk", "Desktop/") if fn was:

fn function (curr,prev) {
  if (curr.lastIndexOf("/") !== prev.lastIndexOf("/")) return true
  else return false
})
```

**When would you want to use Trigger?**

When you use `cd` in Fig, we set a trigger for `/`.

Every time a `/` is encountered, Fig will run the generator again to find the new folders under the new working directory.

## Filter Term

A function to determine what part of the string the user is currently typing we should use to filter our suggestions.

The default is the full string containing the user's input.

```js
filterTerm: (currentString: string): string => {

}
```

**When would you want to use Filter Term?**

e.g. For `cd`, the user may have typed `~/Desktop/b`

- Our trigger function tells us to regenerate suggestions when we see a new `/`
- Our suggestions show all the folders in the `~/Desktop` directory
- As the user types, Fig defaults to filter over the suggestions using `~/Desktop/b` as the search term. We only want the search term to be `"b"`

Here is the function we would write:

```java
filterTerm: (curr) => {
  if curr.includes("/") {
    return curr.slice(curr.lastIndexOf("/") + 1)	
  }
  else return curr
}
```

Because this use case may be common, we have also defined a string that makes this simpler. Here, we filter by everything to the right of the last occurrence of our filterTerm property.

```js
filterTerm: "/"
```

Finally, if you have multiple generators, Fig will only take into account the very last filterTerm function or string you define.

**Note**: If you want to use multiple generators for an argument, and one of the generators is template suggestions, you should make sure to include the template generator first in the generators array and to define your filterTerm function after.