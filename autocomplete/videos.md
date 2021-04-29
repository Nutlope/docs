# Fig Video Tutorials


<!-- ###### Start Header ######## -->
<details open>
<summary style="font-size: 24px">Getting Started</summary>

Create and test your first spec in 1 minute!

<iframe style="border:0;" width="100%" height="450" src="https://app.tella.tv/embed/cko1xfza2000209mlbthpba94" allowfullscreen></iframe>

![Fork withfig/autocomplete](https://img.shields.io/github/forks/withfig/autocomplete?style=social)

Make sure to fork your repo, then run the following
```bash
git clone https://github.com/YOUR_GITHUB_USER_NAME/autocomplete.git
cd autocomplete

# Install packages
npm install

# Create an example spec and choose a name
npm run create-boilerplate

# Go into Figg's testing mode
npm run dev

```
Now save your file, open a new terminal. Voilà! 

When you're finished, hit `ctrl+c `

<br> 


</details>
<!-- End Header -->





<!-- ###### Start Header ######## -->
<details>
<summary style="font-size: 24px">The Fig Completion Spec Standard</summary>


<!-- Start Single Video -->
<details>
<summary>Overview</summary>



#### Key Takeaways


### Fig's Tokenizer
Fig's tokenizer takes the user's input and turns it into an array. At a high level, the tokenizer handles quotes, then splits on spaces, separates chained options, and separates options from their arguments. Examples

* `git push origin master` → `["git", "push", "origin", "master]`
* Quotes: 
    * Double Quotes: `git commit -m "brendan's commit message"` → `["git", "commit", "-m", "brendan's commit message"]`
    * Single Quotes: `git commit -m "hello world"` → `["git", "commit", "-m", "hello world"]`
* Escape Characters: `echo hello\ world 1234` → `["echo", "hello world", "1234"]`
* Options: 
    * Long options `--message="hello"` and `--message "hello"` → `["--message", "hello"]`
    * Short options `-m "hello"`, `-m="hello"` → 
    * Chained short options: `ps -aux` → `["ps", "-a", "-u", "-x"]`
        * Note: Short options that take arguments can also do `-n10` where 10 is the argument for `-n`. Fig will initially tokenize it incorrectly, but as it maps the tokens to the completion spec, it will correct for its mistake. You do not need to worry about this
    * non posix compliant options `ps -aux` → `["ps", "-aux"]`
* Extras:





### Completion Spec Rules
Fig handles all of the annoying argument parsing logic for you automatically. In order for your specs to work properly, you must follow these rules below:

1. **Subcommand Objects**
    1. can nest options, arguments (through the `args` prop), and other subcommands

2. **Option Objects**
    1. Can only nest arguments  (through the `args` prop)
    2. You should not worry about users chaining options - Fig will automatically handle this
        * If the user inputs `-aux`, Fig will know this means `-a` `-u`, `-x` - You should create one option object for each
    3. You should not worry about the different ways users can input option arguments - Fig will automatically handle this
        * Fig knows that `-m"hello"`, `-m "hello"`, `--message="hello"` and `--message "hello"` are all the same - you should just have one option object that nests one argument object

3. The **`name`** prop of subcommand/option objects
    1. Must exactly match what the user has input afte Fig uses the `name` prop for parsing. It is important you get this exactly right
    2. Is mandatory
    3. The `name` prop can be a string or an array, e.g. `"-m"`, `["i", "install"]`, and `["-m", "--message"]` are all valid
    4. The `name` prop should not include the `=` sign, `[arg]`, `<arg>`, or any spaces or special characters - If a subcommand/option takes an argument, include an argument object in the `args` object (see below)
        
3. The **`args`** prop of subcommand/option objects 
    1. If a subcommand/option takes one argument, the `args` prop must take one argument object ie `args: {}` (Note: `{}` is a valid argument object, see point 4)
    2. If a subcommand/option takes multiple arguments, the `args` prop must take an array of argument objects, one for each potential argument type ie if the subcommand/option takes two arguments, then you must put `args: [{}, {}]`
    3. If a subcommand/option object does not take an argument, do not include the `args` prop
4. **Argument Objects**:
    1. Fig treats an empty arg object (ie `{}`) as a mandatory argument
    2. If an argument is optional, the arg object must contain `isOptional: true`
    3. If an argument is variadic (ie it repeats infinitely), the arg object must contain `variadic: true`
    4. The `name` prop of an argument object is not important for Fig's parsing purposes

6. **Other**
    1. Fig handles the `--` end of options separator 
    2. 
 
 ### Things Fig completion spec can handle but it's not optimal
 * Different argument paths:
    * eg kubectl
 
 ### Things Fig's completion spec currently does not handle
 * Mutually exclusive options e.g. in a man page you might see `[-p | -G | -m]`. 
 
    
    

<br />
<p>The below is a perfectly valid spec</p>


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



</details>
<!-- End Single Video -->



<!-- Start Single Video -->
<details>
<summary>Common Mistakes When Building Specs</summary>

#### Key Takeaways
1. 

</details>
<!-- End Single Video -->



</details>
<!-- End Header -->

<!-- Start Header -->
<details>
<summary style="font-size: 24px">Customizing Subcommand & Option Suggestions</summary>
</details>
<!-- End Header -->


<!-- Start Header -->
<details>
<summary style="font-size: 24px">Generating Argument Suggestions (Generators)</summary>
</details>
<!-- End Header -->






<!-- <iframe style="border:0;" width="100%" height="450" src="https://app.tella.tv/embed/cko1xfza2000209mlbthpba94" allowfullscreen></iframe>
[View Code](https://github.com/withfig/autocomplete/tree/master/dev/_examples) -->