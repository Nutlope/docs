# Fig's Tokenizer
Fig's tokenizer takes the user's terminal input and parses it into an array of strings (the tokens). At a high level, the tokenizer handles quotes, then splits on spaces, separates chained options, and separates options from their arguments. Examples

* Standard: `git push origin master` → `["git", "push", "origin", "master]`
* Quotes: 
    * Double Quotes: `git commit -m "brendan's commit message"` → `["git", "commit", "-m", "brendan's commit message"]`
    * Single Quotes: `git commit -m 'my "commit" msg'` → `['git', 'commit', '-m', 'my "commit" msg']`
* Escape Characters: `echo hello\ world 1234` → `["echo", "hello world", "1234"]`
* Options with arguments: 
    * Long options `--message="hello"` and `--message "hello"` → `["--message", "hello"]`
    * Short options `-m "hello"` and `-m="hello"` → `["-m", "hello"]`
    * Short options with arguments not separated by `=` or space: `grep -C50` → `["grep", "-C" "-50"]`
      * Fig will tokenize this correctly with a few small exceptions (see below)
* Chained short options: `ps -aux` → `["ps", "-a", "-u", "-x"]`
  * Chained short options that take an argument not separated by space at end: `grep -abC555` → `["ps", "-a", "-b", "-C", "555"]`
    * Fig will tokenize this correctly with a few small exceptions (see below)
* **Settings** (You can add setting's to Fig's completion object to affect the parsing)
  * Non posix compliant options: `nonPosixCompliance: true`
    * `ps -aux` → `["ps", "-aux"]`
  


## Where Fig's Tokenizer breaks:
* Chained short options where the second option or after is not a lowercase or uppercase letter or the number `1` (1 is surprisingly a common flag e.g. `ls`)
  * e.g. `ls -ga@` will output `["ls", "-g", "-a", "@"]` instead of `["ls", "-g", "-a", "-@"]`
  * e.g. `ls -g%aG` will output `["ls", "-g", "%aG"]` instead of `["ls", "-g", "%", "a", "G"]`
  * BUT 
    * `ls -ga1` will work as expected because Fig consider 1 to be an option
    * `ls -% -@` will work as the `@` and `%` symbols are the first option, not the second
    * `git push -4 -6` will work as `4` and `6` are the first option in each token, not the second
* Short options that take argument that starts with a lowercase letter, uppercase letter, or 1
  * e.g. `grep -C1` will output `["grep", "-C", "-1"]` instead of `["grep", "-C", "1"]`
  * e.g. `grep -C100` will output `["grep", "-C", "-1", "00"]` instead of `["grep", "-C", "100"]`
  * e.g. `git commit -mhello` will output `["git", "commit", "-m", "h", "e", "l", "l", "o"]` instead of `["git", "commit", "-m", "hello"]`
  * BUT 
    * `grep -C51` will work as the 5 will trigger the rest of the token to be considered an argument


<br>
<br>

# Completion Spec Rules
As described above, Fig's tokenizer handles all of the annoying argument parsing logic for you automatically. We take these tokens and map them to the relevant completion spec. This output is then used to generate suggestions. Fig's spec structure is very specific: In order to deliver the right suggestions, your spec must follow these rules below:


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
    * eg kubectl
 
 ### Things Fig's completion spec currently does not handle
 * Mutually exclusive options e.g. in a man page you might see `[-p | -G | -m]`. 
   * For the moment, just don't include this



## Most common completion spec mistakes
* Not including `args: {}` when a subcommand or option takes an argument
* Not including `isOptional` when an argument is optional
* Including a subcommand's argument's inside an option's arguments (keep them separate, Fig will handle the logic)
* Adding extra information to a subcommand/option `name` prop that Fig has already parsed out (e.g. `=`, `[]` or `<>` should NOT go in `name`)
 




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




# Generating Suggestions

Fig takes the tokenized user input and essentially walks down the tree of of the relevant completion spec. It then outputs:
1. The nested subcommand object the user has "walked" down to
   1. A completion spec is effectivel the same as a subcommand. And because subcommand objects are recursive, if I type `git push origin`, the `push` subcommand in the `git` completion spec is essentially its own completion spec
2. The argument object that could potentially be now
   1. Remember Fig handles all of the logic of optional vs mandatory arguments, including if there is a conflict (e.g an option takes an optional argument and a subcommand takes an optional argument)

Given this, Fig can generate suggestions

## Generating Suggestions from the Subcommand Object
Fig takes the subcommand object and automatically turns:
* the array of subcommands in the `subcommands` prop to an array of suggestions objects with `type: subcommand`
* the array of options in the `options` prop to an array of suggestion objects with `type: option`
* the array of suggestions in the `additionalSuggestions` prop to an array of suggestion objects with default `type: special`


## Generating Argument Suggestions

Fig takes the argument object and generates suggestions in one of 4 ways:
1. Templates
2. Script as a string
3. Script as a function
4. Custom Function


To be continued!