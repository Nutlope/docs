# Building Autocomplete for Node Scripts

A Fig autocomplete spec can be bundled into any Node.js project by simply including a Fig object in the project's `package.json`. In this guide, we'll walk through a use case and example of bundling a completion spec into your `package.json`.


## Why add autocomplete to node scripts?
Node projects typically contain multiple scripts that are defined in the `package.json`. For example, a typical React app contains a test script, a build script, and a command to run the built project.

This setup would look like this in your `package.json`.

```json
"scripts": {
    "test": "react-scripts test",
    "build": "react-scripts build",
    "start": "react-scripts start",
},
```

Adding autocomplete serves as a lightweight way to document the npm scripts in your project, both for yourself and for any teammates that have Fig installed. It's great for internal tools that don't require a full blown spec to be written — including autocomplete in your `package.json` makes sure it comes pre-bundled for all Fig users that have access to your project.

By default, Fig automatically generates autocomplete suggestions based on elements in the `"scripts"` object of the `package.json`, but you can override the default suggestions with custom descriptions, icons, and priorities by adding a Fig Object.

## Adding the Fig Object

```json
{
    "name": "react-app",
    "version": "1.0.0",
    "description": "My awesome react app",
    "scripts": {
        "test": "react-scripts test",
        "build": "react-scripts build",
        "start": "react-scripts start",
    },
    "fig": {
        "test": {
            "description": "Run all tests",
            "icon": "🧪",
            "priority": 100
        },
        "build": {
            "description": "Build the project",
            "icon": "🛠",
        },
        "start": {
            "description": "Start the react app",
            "icon": "⭐️"
        }
    }
	...
}
```

Add a separate JSON object with `"fig"` as its key. Under this object, we nest JSON objects, with their keys being the names of your scripts. Suggestions in the Fig object will only show up if a script with the same name is found under the `"scripts"` object. Note that you can include a description, icon, and custom priority property to each script item.

Now, when anyone in this node project's directory enters `npm run` or `yarn run` into their terminal, Fig will present a Suggestion Object for each script defined.

![](/docAssets/autocomplete/scripting/reactExample.png)