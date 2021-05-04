# Argument Object

The subcommand and option object are based on Suggestion objects. Everything you can do to customise a suggestion object, you can do to a subcommand or option object.


The argument object is used to generate suggestions for an argument. For instance, the `cd` command takes one argument: a folder. The arg object for `cd` therefore generates suggestions for all of the folders in a user's current directory. To do this, Fig literally runs `ls` in the user's terminal's current working directory, parses out all the folders, turns them into suggestion objects, then renders them.

There are 4 ways you can use argument objects to generate suggestions. These are more comprehensively outlined in 


COMING SOON