# Completion Spec Rules

At a high level, Fig's completion specs have three important object types: `subcommands`, `options` and `arguments`. 



![cli_structure](/docAssets/autocomplete/handbook/cli_structure.png)



Fig takes the user's input, tokenizes it, and then maps each token to the completion spec. We then predict what will come next. For instance, if the user input a subcommand then an option, what will come next depends on the completion spec. If the option takes an  argument, Fig will only offer suggestions for that argument. If it doesn't take an argument, Fig will offer suggestions for all the other options + the subcommand's first argument.  



Fig's spec structure is very specific: In order to deliver the right suggestions, your spec must follow the rules below:


2. **Subcommand Objects**
   1. Can nest options, arguments (through the `args` prop), and other subcommands

3. **Option Objects**
   1. Can only nest arguments (through the `args` prop)
   2. Only nest arguments that relate to the OPTION, not to the subcommand. Fig will infer when you have finished inputting an option argument (even if the option argument is optional)
      1. e.g. `grep` takes two subcommand arguments (pattern and file). Grep has the option `-A` which takes on argument (the `num`). (pattern and file) are grep's arguments, (num) is `-A`'s argument
   3. Don't worry about chaining options (e.g. `ps -aux`) or the different ways you can input options (e.g. `-m"hello"`, `-m "hello"`, `--message="hello"` and `--message "hello"`) - Fig will handle this in the tokenizer (see above) 

4. The **`name`** prop of subcommand/option objects
   1. The `name` prop must exactly match a token in Fig's tokenizer (above). If Fig can't match a token, it will assume the user input an argument. If we weren't expecting an argument, Fig will simply error.
   2. The `name` prop is mandatory
   3. The `name` prop can be a string or an array, e.g. `"-m"`, `["i", "install"]`, and `["-m", "--message"]` are all valid
   4. The `name` prop should not include the `=` sign, `[arg]`, `<arg>`, or any spaces or special characters. Remember, Fig parses all of this out in the tokenizer above.If a subcommand/option takes an argument, include an argument object in the `args` object (see below)

5. The **`args`** prop of subcommand/option objects 
   1. If a subcommand/option takes one argument, the `args` prop must take one argument object ie `args: {}` (Note: Althought `{}` is an empty object, it is a valid argument object, see Argument Objects section)
   2. If a subcommand/option takes multiple arguments, the `args` prop must take an array of argument objects, one for each potential argument type ie if the subcommand/option takes two arguments, then you must put `args: [{}, {}]`
   3. If a subcommand/option object does not take an argument, do not include the `args` prop
6. **Argument Objects**:
   1. Fig treats an empty arg object (ie `{}`) as a mandatory argument
   2. If an argument is optional, the arg object must contain `isOptional: true`
   3. If an argument is variadic (ie it repeats infinitely), the arg object must contain `variadic: true`
   4. The `name` prop of an argument object is not important for Fig's parsing purposes

7. **Other**
   1. Fig handles the `--` end of options separator for you. Do not include it in the spec


 ### Things Fig completion spec can handle but it's not optimal

 * Different argument paths:
   * eg in `kubectl` you can often input TYPE/NAME or TYPE NAME. Fig has no way of distinguishing which "path" the user has chosen. Therefore you must assume the user could input the max number of arguments for any given path and make the 2nd or great argument optional
     * `args: [{name: "TYPE | TYPE/NAME"}, {name: "NAME", isOptional: true}]`

 ### Things Fig's completion spec currently does not handle

 * Mutually exclusive options e.g. in a man page you might see `[-p | -G | -m]`. 
   * For the moment, just don't include this. 



## Most common completion spec mistakes

* Not including `args: {}` when a subcommand or option takes an argument
* Not including `isOptional` when an argument is optional
* Not including `variadic` when an argument is variadic 
* Including a subcommand's argument's inside an option's arguments (keep them separate, Fig will handle the logic)
* Adding extra information to a subcommand/option `name` prop that Fig has already parsed out (e.g. `=`, `[]` or `<>` should NOT go in `name`)
  * ❌ `{ name: "--flag=parameter" }`
  * ✅ `{ name: "--flag", args: {name: parameter" } }`




### Mini Example of a perfectly valid spec


```typescript
export const completionSpec: Fig.Spec = {
  name: "git"
  subcommands: [
      {
          name: "checkout"
          args: {}  // git checkout takes one mandatory option
      },
      {
          name: "push"
          args: [{ isOptional: true }, { isOptional: true, variadic: true }]  // git push takes two optional arguments. The second repeats infinitely
      },
      {
          name: "add"
          args: { variadic: true }  // git add takes one argument that repeats infinitely
      },
      {
          name: "add"
          args: { variadic: true }  // git add takes one argument that repeats infinitely
      },
  ]
}

```





## Man Pages & `--help` text



### Understanding the Synopsis section of man pages 

This section is copied straight from [https://github.com/docopt/docopt](https://github.com/docopt/docopt)

- **<arguments>**, **ARGUMENTS**. Arguments are specified as either upper-case words, e.g. ``my_program.py CONTENT-PATH`` or words surrounded by angular brackets: ``my_program.py <content-path>``.
- **--options**.  Options are words started with dash (``-``), e.g.``--output``, ``-o``.  You can "stack" several of one-letter options, e.g. ``-oiv`` which will be the same as ``-o -i -v``. The options can have arguments, e.g.  ``--input=FILE`` or ``-i FILE`` or even ``-iFILE``.
- **commands** are words that do *not* follow the described above conventions of ``--options`` or ``<arguments>`` or ``ARGUMENTS``, 

Use the following constructs to specify patterns:

- **[ ]** (brackets) **optional** elements.  e.g.: ``my_program.py [-hvqo FILE]``
- **( )** (parens) **required** elements.  All elements that are *not* put in **[ ]** are also required, e.g.: ``my_program.py --path=<path> <file>...`` is the same as ``my_program.py (--path=<path> <file>...)``.  (Note, "required options" might be not a good idea for your users).
- **|** (pipe) **mutually exclusive** elements. Group them using **( )** if one of the mutually exclusive elements is required:``my_program.py (--clockwise | --counter-clockwise) TIME``. Group them using **[ ]** if none of the mutually-exclusive elements are required: ``my_program.py [--left | --right]``.
- **...** (ellipsis) **one or more** elements. Specify that elements are variadic ie an arbitrary number of repeating elements could be accepted, use ellipsis (``...``), e.g.  ``my_program.py FILE ...`` means one or more ``FILE``-s are accepted.  If you want to accept zero or more elements, use brackets, e.g.: ``my_program.py [FILE ...]``. Ellipsis works as a unary operator on the expression to the left.
- **[options]** (case sensitive) shortcut for any options.  You can use it if you want to specify that the usage pattern could be provided with any options defined below in the option-descriptions and do not want to enumerate them all in usage-pattern.
- **Special:**
  - "``[--]``". Double dash "``--``" is used by convention to separate positional arguments
    - NOTE: Fig will handle this automatically
  - "``[-]``". Single dash "``-``" is used by convention to signify that ``stdin`` is used instead of a file
    - NOTE: You should not handle this, Fig will treat it as a file input



### Converting Synopsis from man pages to Fig Spec

In your head think:

1. Is this value an option or an argument? 
2. If it's an argument
   1. Is its parent an option or a subcommand?
   2. Is it an optional argument? `isOptional: true` 
   3. Is it a variadic argument (repeats infinitely)? → `variadic: true`



#### Arguments

* `[arg]` → optional argument

* `<arg>` → mandatory argument

* `[<arg>]` → optional argument

* `[<arg1> [arg2]]` → both args are optional

* `[<arg1> [<arg2>]]` → both args are optional
* `[<arg1> <arg2>]` →  arg1 is optional, arg2 is mandatory
  * Why? Inputting arguments is optional. But as soon as you input the first one, you *must* input the second one
* ` <arg...>` → Mandatory arg that is variadic
* ` [arg...]` → Optional arg that is variadic
* `[<arg1> [<arg2>...]` → arg1 is optional, arg2 is optional and variadic
* `[<arg1> [<arg1>...]` → arg1 is optional and variadic
  * This syntax happens in man pages regularly.The same argument repeats twice but the second is variadic... In this case, just include **one** argument and then make it variadic. Do NOT include two arguments
* `[<arg1> [<arg1>...]]` → arg1 is optional and variadic
* `[<refname>[:<expect>]]` → This is just **one** argument that is optional. Let's break this down
  * This argument is optional as it is surrounded by `[]`
  * Inside the argument we have `[:<expect>]` → The square brackets around `[:<expect>]` indicate that, if there is a `:`, the user can optionally input a second argument afterwards called `expect` **within the same string**. Because this second argument is part of the same string, Fig's tokenizer would treat this argument as the **one** token.
  *  **Therefore, as Fig's tokenizer treats it as one token, the completion spec must treat it as one argument**
    * e.g. a user could  type `abc:def` for this argument, Fig's tokenizer would treat it as `[..., "abc:def"]` 
  * We would give this argument `name: "refname[:expect]"`
  * Can Fig still offer suggestions for the `expect` part of this argument?
    * Yes. Fig can offer special suggestions for the the `expect` section of the argument after the user inserts a `:` even though the completion spec says it only has one argument. 
    * You will need to look into `trigger`  and script as a function in the Generator objects and  `filterTerm` in the Argument object.
* `[ <arg1...> [arg2...]` → TRICK! This shouldn't exist. Because arg1 is variadic we will never get to arg 2. Also note that that arg1 is optional



#### Options

* `--atomic` → an option with `name: "--atomic"`
* `[--atomic]` → an option with `name: "--atomic"`
  * Although it is surrounded by square brackets indicating that it is optional, Fig currently doesn't support a syntax for "optional arguments"
* `[-ABCFGHLOPRSTUW@abcdefghiklmnopqrstuwx1%]` → This is a list of options. The first option has `name: "-A"`, the second option has `name: "-B"` etc
  * Note: Users can chain options together like this, however, Fig will handle it in its parser
* `[-n | --dry-run]` → an option with `name: ["-n", "--dry-run"]`
  * A pipe separating a short option (one dash) and a long option (two dashes) usually means the options are the same but can be referred to in either way
* `[--all | --mirror | --tags]` → three separate options
  * A pipe separating multiple long options or multiple short options usually indicates they are mutually exclusive

* `[-o <string>]`→ an option with `name: "-o"` that takes a mandatory argument with `name: string`
* `[--receive-pack=<git-receive-pack>]`→ an option with `name: "receive-pack"` that takes a mandatory argument with `name: git-receive-pack`
  * **Note**: Do not worry about the `=` sign. Fig handles this in its Tokenizer
* `[--[no-]signed]` → This is actually shorthand for `[--signed] [--no-signed]` and so it simply two options:
  * The first has `name: "--no-signed"`
  * The second has `name: "--signed"`
* `[-S[<keyid>]]` → this is an option with `name: "-S"` with optional argument with `name: "keyid"`
  * Why? Short options (options with one `-`) sometimes allow you to insert an argument without a space... e.g. `git commit -m"Hello"`
  * Fig's parser will handle this special logic
* `[--chmod=(+|-)x]`→ this is an option with `name: "chmod"` and one mandatory argument with `name: "x"`
  * Note: the `()` indicate that you can precede the argument with a `+` or `-`. This should not be included in the spec. You can do custom suggestions here with trigger, filterTerm and script as a function.
* `[--force-with-lease[=<refname>[:<expect>]]]` → Let's break this down
  * This is an option with `name: "--force-with-lease"`
  * It takes **one** argument with `name: "refname[:expect]"`
    * This argument is optional `[=<refname>[:<expect>]]` as it is surrounded by square brackets
    * Then we would parse the argument the same as we did in this example in the Arguments section above
* `[--signed|--signed=(true|false|if-asked)]` → This is an option with `name: "--signed"` that takes an optional argument with `name: "SIGNED"`
  * The `()` indicate that these are the possible arguments for the `--signed` option. However, because we have the pipe symbole, clearly `--signed` can exist on its own (ie without any options). Therefore, the argument is optional
  * An additional note: because we have a defined list of suggestions here, you could also give the argument object the `suggestion: ["true", "false", "if-asked"]` and Fig will generate these as suggestions for it.

* Annoyingly tricky:
  * `[--[no-]signed|--signed=(true|false|if-asked)]` → Let's break this down with everything we learned above
  * The pipe symbol indicates mutually exclusive (above) meaning  `--signed` takes an optional argument
  * the `[no-]` indicates shorthand (above) meaning `--no-signed` and `--signed` are valid options



### Other

* `[<options>]`, `[options]`, or `<options>` → indicate that this is where a spec would insert their options
* `[<command>]`, `[command]`, or `<command>` → indicate that this is where a spec would insert their subcommands
* `[<args>]`, `[args]`, or `<args>` → indicate that this is where a spec would insert their arguments







### Examples of Converting Common CLIs to Fig's standard

```typescript

SYNOPSIS
       git push [--all | --mirror | --tags] [--follow-tags] [--atomic] [-n | --dry-run] [--receive-pack=<git-receive-pack>]
                  [--repo=<repository>] [-f | --force] [-d | --delete] [--prune] [-v | --verbose]
                  [-u | --set-upstream] [-o <string> | --push-option=<string>]
                  [--[no-]signed|--signed=(true|false|if-asked)]
                  [--force-with-lease[=<refname>[:<expect>]]]
                  [--no-verify] [<repository> [<refspec>...]]

                  
Coming soon!!
                                 
In the meantime, check out `git push --help` and the git completion spec


```

