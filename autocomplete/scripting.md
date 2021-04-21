# Adding autocomplete to npm and yarn scripts

## Autocomplete for npm and yarn projects

You can add a Fig object to your package.json, which will allow Fig to automatically generate autocomplete suggestions when using your project.

Including the Fig object can serve as a quick reference guide for your project's CLI commands. This can be useful when sharing a node-based internal tool with team members who all have Fig installed.

```json
{
  "name": "autocomplete",
  "version": "1.0.0",
  "description": "Fig Autocomplete Spec Linter",
  "fig": {
    "dev": {
      "description": "Watching and compile .ts files in ./dev",
      "icon": "fig:*//template?badge=🛠 ",*
      "priority": 100
    },
    "create-boilerplate": {
      "description": "Create a new completion spec"
    },
    "lint:fix": {
      "description": "Fix linting issues"
    },
    "lint": {
      "description": "Check for linting issues"
    },
    "build": {
      "description": "Compile all files in /dev"
    },
    "lint": {
      "description": "Typecheck all .ts files in /dev"
    }
  }
	...
}
```

The above Fig object in the `package.json` generates these autocomplete suggestions when typing `npm run`.

![](/docAssets/autocomplete/scripting/packagejson.png)


## The Fig Object

The Fig object is denoted with `"fig": {}` anywhere in your `package.json`. Every directly nested object is equivalent to a [Suggestion Object](/docs/api#Subcommand-Object), which could include a `description`, `icon`, `priority`, etc. The key of the object is equivalent to the `name` property of the Suggestion Object.

