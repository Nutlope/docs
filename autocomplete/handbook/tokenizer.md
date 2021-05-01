# Fig's Tokenizer
Fig's tokenizer takes the user's terminal input and parses it into an array of strings (the tokens). At a high level, the tokenizer handles quotes, then splits on spaces, separates chained options, and separates options from their arguments. Examples



| User Input                                                   | Tokenized Output                                            |
| ------------------------------------------------------------ | ----------------------------------------------------------- |
| **Standard**<br /><br />`git push origin master`<br />       | <br /><br />  `["git", "push", "origin", "master]`          |
| **Quotes**                                                   |                                                             |
| Double Quotes<br />`git commit -m "brendan's commit message"` | <br />`["git", "commit", "-m", "brendan's commit message"]` |
| Single Quotes<br />`git commit -m 'my "commit" msg'`         | `['git', 'commit', '-m', 'my "commit" msg']`                |
| **Escape Characters**<br /><br />`echo hello\ world 1234`    | <br /><br /> `["echo", "hello world", "1234"]`              |
| **Options with Arguments**                                   |                                                             |
| Long options<br /> `--message="hello"` and `--message "hello"` <br /><br /> | <br />`["--message", "hello"]`                              |
| Short options<br /> `-m "hello"` and `-m="hello"`            | <br />`["-m", "hello"]`                                     |
| Short options with arguments not separated by `=` or space<br />`grep -C50` <br /><br />*See exceptions to this below* | <br />`["grep", "-C" "-50"]`                                |
| **Chained short options**                                    |                                                             |
| Normal chained options<br /> `ps -aux`                       | <br />`["ps", "-a", "-u", "-x"]`                            |
| Chained options with argument at end<br /><br />`grep -abC555` → <br />*See exceptions to this below* | <br />`["grep", "-a", "-b", "-C", "555"]`                   |
| **Settings**<br />You can add settings to Fig's completion spec to affect the tokenizer |                                                             |
| Non posix compliant options `posixNoncompliantFlags`<br />`ps -aux` | <br /> `["ps", "-aux"]`                                     |





## Where Fig's Tokenizer breaks:

- Chained short options where the second option or after is not a lowercase or uppercase letter or the number `1` (`1` is surprisingly a common flag `ls`)
  - e.g. `ls -ga@` will output `["ls", "-g", "-a", "@"]` instead of `["ls", "-g", "-a", "-@"]`
  - e.g. `ls -g%aG` will output `["ls", "-g", "%aG"]` instead of `["ls", "-g", "%", "a", "G"]`
  - BUT
    - `ls -ga1` will work as expected because Fig consider 1 to be an option
    - `ls -% -@` will work as the `@` and `%` symbols are the first option, not the second
    - `git push -4 -6` will work as `4` and `6` are the first option in each token, not the second
- Short options that take argument that starts with a lowercase letter, uppercase letter, or 1
  - e.g. `grep -C1` will output `["grep", "-C", "-1"]` instead of `["grep", "-C", "1"]`
  - e.g. `grep -C100` will output `["grep", "-C", "-1", "00"]` instead of `["grep", "-C", "100"]`
  - e.g. `git commit -mhello` will output `["git", "commit", "-m", "h", "e", "l", "l", "o"]` instead of `["git", "commit", "-m", "hello"]`
  - BUT
    - `grep -C51` will work as the 5 will trigger the rest of the token to be considered an argument