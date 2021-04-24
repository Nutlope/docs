# Using Different Generator Types

### Types of Generators

There are 4 types of generators. They all give you the ability to define what shell command(s) to run. Then given that output, parse it however you'd like to produce Suggestion objects.

They types of Generator are listed below in order of customisability.

We would say the vast majority of Generators you make will be using **Templates** and/or **Script as a String**

For reference on each generator type's properties, see the [Generators Reference](/docs/autocomplete/reference/generators)

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
```

Including a `filterTemplateSuggestions` function allows for direct filtering of the suggestions generated by the `template`. In this example, only JavaScript files are shown to the user.

```js
// Define templates directly underneath args if you don't need to filter
// This also looks nicer and is easier to read
args: {
  template: "filepaths",
}
```


### Script as a String

If you're suggesting something other than files or folders, we recommend using this. This let's you run a mini-script / command on the user's computer. The script runs in the user's current working directory and in the same shell. It's as if the user typed the command themselves.

The output from the script you specify is then converted to Suggestion objects. We can do this for you with `splitOn` or you can process the output yourself with `postProcess`. Not that you must specify at least one of `splitOn` or `postProcess`.

```js
var generateBranches = {
  script: "git branch",
  splitOn: "\\n",
  ** OR **
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
}
```

The above code runs the `git branch` script and generates a list of suggestion objects by splitting the output by newline characters with the `splitOn` function. Alternatievly, more complex filtering logic can be written in the `postProcess` method, which takes the output of the script as an output, and returns an array of suggestion objects.

### Script as a Function

Using Script as a Function may be necessary when you want to run scripts dynamically based on parts of the command that the user has entered. For example, Heroku requires the function takes a parameter, `context`, which includes an array of strings broken down into components. The strings can then be used as values to pass to the script you'd like to return.

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

### Custom Function

1. An argument can be one of many types, and you want to provide suggestions once you have more context on what type the user is inputting.

e.g. `git checkout` supports multiple types of arguments

- `git checkout staging` → branch
- `git checkout h8ne3x` → commit hash
- `git checkout HEAD~` → Relative refspec

(If you're interested [here are a lot more](https://stackoverflow.com/a/18605496/2218728))

You could just use a **script as a string** Generator and only suggest branches by using `git branch`. This would be nice and would work fine. But it would be cool if you could type `HEAD^^` and have Fig tell you which commit description you are referring to. Custom functions let you do this.

2. An argument takes its own sub-arguments that are not delimited by a space

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
    }
  }	
}
```