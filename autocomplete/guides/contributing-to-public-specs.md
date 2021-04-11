# Contributing to Public Specs

If you're looking to improve existing autocomplete functionality, or to add support for a new CLI tool, you can contribute to the [autocomplete repo](https://github.com/withfig/autocomplete).

## Add a completion spec for a CLI tool 

Completion specs are defined in a *declarative* schema that specifies `subcommands`, `options` and `arguments`. Suggestions can be generated dynamically by running shell commands or reading local files, in addition to the information in the spec itself.

**For more documentation and tutorials**, visit [withfig.com/docs](https://withfig.com/docs/autocomplete/getting-started)

**To request completions for a CLI tool**, open an [issue](https://github.com/withfig/autocomplete/issues/new/choose).


## 1. Set up the Autocomplete Repo

Fork the [autocomplete repo](https://github.com/withfig/autocomplete) on Github, and clone it to your computer. Then, install the required npm packages.

```bash
git clone [fork git url] fig-autocomplete

cd fig-autocomplete

# Install packages
npm install

# Create a new branch
git checkout -b myBranchName
```

Make sure you're working on a new branch!


## 2. Develop your spec

To develop a spec, we need to first enter testing mode. From the autocomplete folder, run the following command:

```bash
# Go into testing mode
npm run dev
```

In your autocomplete folder, there will be a `specs/` folder and a `dev/` folder. When making changes, make sure to edit the TypeScript files in the dev/ folder.

When testing mode is on, all changes made in the `dev/` folder will automatically compile to the `specs/` folder. Test your spec's functionality directly in the terminal.

**Note**: Fig usually looks for completion specs in your `~/.fig/autocomplete` folder. When in testing mode, we check your cloned repo's `specs/` folder


## 3. Test your spec
To make sure all the specs are of the right format, we can lint all specs in the `dev/` folder by running the following:

```bash
npm test
```



## 4. Commit your spec, and make a pull request

```bash
git add .
git commit -am "commit message"
git push origin myBranchName
```

Make a pull request on the [autocomplete repo](https://github.com/withfig/autocomplete). Your PR will be reviewed and merged when ready!

## Other available commands
```bash

# Create a new spec from a boilerplate template
npm run create-boilerplate

# Typecheck all specs in the dev/ folder
npm test

# Compile typescripts specs from dev/ folder to specs/ folder
npm run build

# Copy all specs from the specs/ folder to the ~/.fig/autocomplete folder
npm run copy:all

# Copy an individual spec from the specs/ folder to the ~/.fig/autocomplete folder
npm run copy <spec-name>
```