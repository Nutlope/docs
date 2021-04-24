# Building Your First Autocomplete Spec

In this section, you'll learn the autocompletion spec skeleton, and understand how to extend the format to any other CLI tool. To demonstrate, we'll build a basic `git` autocompletion spec that includes support for a few [subcommands](/docs/autocomplete/api#subcommand-object), [arguments](/docs/autocomplete/api#arg-object), and [options](/docs/autocomplete/api#option-object). 

#### 1. Defining the spec

To start, let's define the `completionSpec` variable, and add a `name` and `description`. Fig uses the `name` property when figuring out which spec to load. In this example, when the user enters "git" into their terminal, our completionSpec will be loaded. The name property should match the file name.

Make sure to define your spec exactly as shown, using `var` instead of `const` or `let`. `completionSpec` is case sensitive!

```ts
export const completionSpec: Fig.Spec = {
	name: "git",
  description: "the stupid content tracker"
}
```

#### 2. Adding a subcommand

Now that we have our spec defined, let's add autocomplete support for `git checkout`, so Fig can suggest `checkout` when the user types in `git`.

`checkout` is a subcommand of `git`, so we'll include it under `git`'s subcommands array. The subcommand object takes a name, description, as well as its own options and arguments. For more information on subcommand's properties, see [Subcommand Object](/api#subcommand-object).

```ts
export const completionSpec: Fig.Spec = {
  name: "git",
  description: "the stupid content tracker",
  subcommands: [
    {
      name: "checkout",
      description: "Switch branches or restore working tree files"
    }
  ]
}
```

#### 3. Adding options and args

`git checkout` can also take an option or an argument. Now, we'll add support for the `-b` option flag, and also notify Fig to expect an argument following `checkout`. 

Under the `checkout` subcommand, we added an empty args object. Including this argument object signals to Fig that there checkout potentially takes an argument.

`-b` is an option of the `checkout` subcommand, so we'll nest the flag under the subcommand's `options` array. Options take a name and description, both of which will show in the Fig UI.

Nested under the `-b` option is another argument named branch. The Fig parser won't function properly if args aren't included when there should be user input, so don't forget to at least insert an empty `args: {}` property when there should be an argument.

```ts
export const completionSpec: Fig.Spec = {
  name: "git",
  description: "the stupid content tracker",
  subcommands: [
    {
      name: "checkout",
      description: "Switch branches or restore working tree files",
      args: {},
      options: [
        { 
          name: ["-b"], 
          description: "create and checkout a new branch", 
          args: { 
            name: "branch"
          } 
        },
      ]
    }
  ]
}
```

#### 4. Including an option to the root object

We now have a spec that supports the primary functionality of `git checkout`. The root object, defined as `completionSpec`, is actually a command in itself, with the same properties as the [Subcommand Object](/api#subcommand-object).

If we want to add support for `git --version`, the `--version` flag can be added as an option under the root object as follows:

```ts
export const completionSpec: Fig.Spec = {
  name: "git",
  description: "the stupid content tracker",
  subcommands: [
    {
      name: "checkout",
      description: "Switch branches or restore working tree files",
      args: {},
      options: [
        { 
          name: ["-b"], 
          description: "create and checkout a new branch", 
          args: { 
            name: "branch"
          } 
        },
      ]
    },
  ],
  options: [
    {
      name: ["--version"],
      description: ["View your current git version"]
    }
  ]
}
```



## Making Advanced Suggestions

Above, we put together a spec that supports `git checkout`, and `git --version`. But by using some of Fig's advanced features, we can improve autocomplete even more for `git checkout`.

### Using Generators

#### 1. Generating branches

When a user runs `git checkout`, they typically want to move to another local branch. In our current configuration, Fig is expecting an argument in the form of the empty args object we added to checkout. Fig doesn't provide any suggestions by default when an empty args object is provided.

```js
{
  name: "checkout",
  description: "Switch branches or restore working tree files",
  args: {},
}
```

But, we can generate a list of suggestions that show the user all their local git branches using [Generators](./generators).

A generator runs a script in the user's terminal, and Fig will capture the output of that script. We want to run `git branch --no-color` to have git display the list of branches. 

![](/docAssets/autocomplete/getting-started/generatorScript.png)

The `postProcess` function captures the output from the terminal as a string, and within the function you can write logic to convert the output string into suggestions. We want to split our list of branches by the newline character, and convert each line containing a branch into a [Suggestion Object](#Suggestion Object) containing a `name` and `description`. Note that `postProcess` should return an array of generated suggestion objects.

```js
var branches = {
  script: "git branch --no-color",
  postProcess: function (out) {
    if (out.startsWith("fatal:")) {
      return []
    }
    return out.split('\n').map((branch) => {
      return { name: branch.replace("*", "").trim(), description: "branch" }
    })
  }
}
```

#### 2. Passing the generator into the completion spec

The last step is to pass our new generator into the spec. Because we want the generator to get the list of branches following `git checkout`, we pass `branches` into the `checkout` subcommand's args. 

```js
var branches = {
  script: "git branch --no-color",
  postProcess: function (out) {
    if (out.startsWith("fatal:")) {
      return []
    }
    return out.split('\n').map((branch) => {
      return { name: branch.replace("*", "").trim(), description: "branch" }
    })
  }
}

export const completionSpec: Fig.Spec = {
  name: "git",
  description: "the stupid content tracker",
  subcommands: [
    {
      name: "checkout",
      description: "Switch branches or restore working tree files",
      args: {
        generators: branches
      },
      options: [
        { 
          name: ["-b"], 
          description: "create and checkout a new branch", 
          args: { 
            name: "branch"
          } 
        },
      ]
    },
  ],
  options: [
    {
      name: ["--version"],
      description: ["View your current git version"]
    }
  ]
}
```

Our one branch, main, now shows as an autocomplete suggestion.

![](/docAssets/autocomplete/getting-started/generatedBranches.png)



For more details about writing generators, see [Generators](./generators).

### Using argument templates

#### 1. Suggesting files and folders

Fig has a few commonly used generators built-in as templates.

Let's improve our `git` completion spec by adding support for `git add`. When running the command, users can select individual files and folders. We can have Fig show suggestions for possible files and folders to add using the "filepaths" template.

We include the `add` subcommand under `checkout`, which accepts one argument. Entering the "filepaths" template under that argument automatically suggests files from the user's working directory when `git add` is typed into the terminal.

```js
var branches = {
  script: "git branch --no-color",
  postProcess: function (out) {
    if (out.startsWith("fatal:")) {
      return []
    }
    return out.split('\n').map((branch) => {
      return { name: branch.replace("*", "").trim(), description: "branch" }
    })
  }
}

export const completionSpec: Fig.Spec = {
  name: "git",
  description: "the stupid content tracker",
  subcommands: [
    {
      name: "checkout",
      description: "Switch branches or restore working tree files",
      args: {
        generators: branches
      },
      options: [
        { 
          name: ["-b"], 
          description: "create and checkout a new branch", 
          args: { 
            name: "branch"
          } 
        },
      ]
    },
    {
      name: "add",
      description: "Stage files to commit",
      args: {
        template: "filepaths"
      }
    }
  ],
  options: [
    {
      name: ["--version"],
      description: ["View your current git version"]
    }
  ]
}
```

Here's what the filepaths template looks like in action:

![](/docAssets/autocomplete/getting-started/template.png)

---

The fastest way to learn is by looking at examples. We suggest looking at some of the completion specs for commands you know well, like [git](https://github.com/withfig/autocomplete/blob/master/specs/git.js) or [npm](https://github.com/withfig/autocomplete/blob/master/specs/npm.js). You should be able to pick up the format pretty quickly.

**All current specs**: [withfig/autocomplete](https://github.com/withfig/autocomplete)

### Next up
Ready to test your spec and put it into action? See [Testing Autocomplete](/docs/autocomplete/testing-autocomplete)