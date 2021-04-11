# Building Autocomplete for Internal Scripts

Fig supports building autocomplete for internal scripts (executables) like `deploy.sh`, `tests.py`, or `build.js`. Assuming these scripts each take different arguments, subcommands, and flags, Fig can be used to build autocomplete for these scripts to make it easy for your team to use them.


## Use the Autocomplete Boilerplate Template

The boilerplate repo contains a copy of our autocomplete repo, but without the generated specs. This can be used to write and test an internal spec, as well as be the files for an internal CLI hosted on an internal repo.

Head to our boilerplate template repo, and click "Use this template" to make a copy of the repo.
https://github.com/withfig/autocomplete-boilerplate

![](/docAssets/autocomplete/building-internal-clis/usetemplate.png)


## Working From The Autocomplete Repo

Clone down your new templated repo to your local machine.

```bash
git clone [git url] [path]
```

When developing a spec, turn on Fig development mode by running `npm run dev` from your autocomplete folder. This should generate a `dev/` folder in the root of your autocomplete folder. 

Development mode compiles all changes from the `dev/` folder into specs that can be read by Fig, and tells Fig to load specs from the `specs/` folder. (If you don't see `specs/` yet, it will be generated upon making changes to any file in `dev/`) This means you can test your internal CLI from your terminal in real time!

Create a TypeScript file in the `dev/` folder for the internal command you're building for. Write your spec in this folder, and make changes in `dev/`. All changes to files in `dev/` will be compiled to `specs/` automatically.

We write our specs in the autocomplete folder to get access to linting and live testing. Now, we need to move the compiled specs from `/specs` into your scripts folder for usage.

## Setting Up Autocomplete for Your Script
In the same directory as your scripts, create a `.fig` directory. The `.fig` directory should contain an autocomplete spec for each script you want to generate suggestions for.

From the autocomplete directory, take the compiled specs from `specs/` and move them into the `.fig` directory of your scripts directory.

Note: the naming of these specs is important! They should exactly match `[script].js`. If you wanted to write a spec for `deploy.py`, name it `deploy.py.js`.

The folder hierarchy should look like this:
```bash
|-- dev.sh
|-- deploy.py
|-- tests.js
|-- .fig/
    |-- dev.sh.js
    |-- deploy.py.js
    |-- test.js.js
```

Your scripts directory containing `.fig` can be shared with teammates, who will automatically get autocomplete for these scripts if they have Fig installed. Want to get your teammates onboard? Run `fig invite` in your CLI!

## Up Next
Looking to build comprehensive internal CLIs for your team with Fig? See [Building Autocomplete for Internal CLIs](/docs/autocomplete/guides/building-internal-clis).

For more information on building more complex autocomplete specs, see [Building Your First Autocomplete Spec](/docs/autocomplete/guides/building-first-spec) and [API](/docs/autocomplete/api).