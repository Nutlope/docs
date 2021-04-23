# Generators

## Quick Summary

Generators let you run shell commands on the user's device to *generate* suggestions for arguments

- There are 4 types of generators (see below)
- Fig runs all shell commands in a separate login-shell at the current working directory of the user's terminal
  - [Executing Shell Commands](./executing-shell-commands)
- Generators are defined in the `generators` property of an Argument object.
  - The `generators` property can be a single Generator object or an array of Generator objects.
- To learn when to use which type of generator and see examples of each generator type, see [Using Different Generator Types](/docs/autocomplete/advancedGuides/using-different-generator-types)

### Generator Usage Examples

- `cd [folder]`
  - `cd` takes one argument: a single folder
    - Arg object 1: A generator that runs  `ls` to get the list of folders at the user's current directory
- `git push [remote] [branch]`
  - `git push` takes two arguments: a remote repo and a branch
    - Arg object 1: Include a generator that runs `git remote` as a script
    - Arg object 2: Include generator that runs `git branch` as a script

### Templates

Because it's so common, Fig has predefined these arguments for you as a template.

For usage, see [Using Different Generator Types](/docs/autocomplete/advancedGuides/using-different-generator-types#templates)

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

You can enter a terminal command as a string, and Fig will output the contents of the terminal after the command is run. This output can be used to generate suggestions.

For usage, see [Using Different Generator Types](/docs/autocomplete/advancedGuides/using-different-generator-types#script-as-a-string)

**Properties**

| Property    | Data Type                                                    | Required | Description                                                  |
| ----------- | ------------------------------------------------------------ | -------- | ------------------------------------------------------------ |
| script      | string                                                       | ☐        | The shell command to execute in the user's current working directory. The output is a string. It is then converted into an array of Suggestion objects using `splitOn` or `postProcess`<br /><br /> e.g. git branch → show all the remote git repos available |
| postProcess | function(scriptOutput: string): SuggestionObject { } <br /><br />**Input**: output of executing script <br />**Output**: array of Suggestion objects | ☐        | **[Must specify one of postProcess or splitOn]** <br /><br />Define a function that takes a single input: the output of executing script. This function then return an array of Suggestion objects that will be rendered by Fig.  <br /><br />**Examples** <br />If `script` outputs data separated cleanly by newlines, or commas... <br />- split the input on new lines <br />- map or filter over your new array <br />- if each line had columns, split it again during the map<br />If `script` outputs json data,  use JSON.parse.<br /><br />**Other ideas** - use try/catch and return an empty array if there's an error |
| splitOn     | string                                                       | ☐        | As splitting the output of `script` is such a common use case for `postProcess`, we build the `splitOn` property. Simply define a string to split the output of script on (e.g. `","` or `"\n"` and Fig will do the work of the `postProcess` prop for you.  <br /><br />This is a simple and fast method to generate suggestions. If you want any more control over the parsing (like filtering) or you want more control over the UI, use `postProcess`. <br /><br />e.g. if the script was `git branch`, specifying splitOn = \n would split the output on new lines and automatically create a Suggestion object for each element. |
| trigger     | function                                                     | ☐        | See [Trigger](#Trigger)                                      |
| filterTerm  | function or string                                           | ☐        | See [Filter Term](#filter-term)                              |

**Notes & Hints**

- In `script`: You can chain multiple commands together in the one string and then use javascript to combine  their output. e.g. `echo "The quick"; echo "brown fox";`
- If a command goes to a pager (ie it scrolls instead of printing out normally) you can try piping it into `cat` e.g. `man git | cat`

### Script as a Function

This is *exactly* the same as **Script as a String**, except you can define what script to run as a **Function**.

For usage, see [Using Different Generator Types](/docs/autocomplete/advancedGuides/using-different-generator-types#script-as-a-function)

**Properties**

| Property    | Data Type                                                    | Required | Description                                                  |
| ----------- | ------------------------------------------------------------ | -------- | ------------------------------------------------------------ |
| script      | function(context: array): string { }<br /><br /> **Input**: output of executing script <br />**Output**: array of Suggestion objects | ☐        | The shell command to execute in the user's current working directory. The output is a string. It is then converted into an array of Suggestion objects using `splitOn` or `postProcess`<br /><br /> e.g. git branch → show all the remote git repos available |
| postProcess |                                                              | ☐        | See [postProcess above](#script-as-a-string)                 |
| splitOn     |                                                              | ☐        | See [splitOn above](#script-as-a-string)                     |
| trigger     | function or string                                           | ☐        | See [Trigger](#trigger)                                      |
| filterTerm  | function or string                                           | ☐        | See [Filter Term](#filter-term)                              |

### Custom Function

Fig's completion spec standard accounts for the vast majority of CLI tools, but some features don't work with the standard spec. Custom Functions inside generators enable you to write more complex logic and generate suggestions based on user input unique to a certain CLI tool.

Custom Functions let you define a function that takes an array of the user's input, run multiple shell commands on the user's machine, and then generate suggestions to display. When using Custom Functions, you will likely have to work pretty closely with [Triggers](#trigger).

For usage, see [Using Different Generator Types](/docs/autocomplete/advancedGuides/using-different-generator-types#custom-functions)

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