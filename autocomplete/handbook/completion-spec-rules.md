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
   1. Fig handles the `--` end of options separator 


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







