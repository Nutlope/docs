# Customizing Priorities in Specs

Fig users can locally configure a setting to present suggestions sorted by recency of use, or by alphabetical order. As a spec developer, you also have control over what suggestions show up first in the autocomplete menu.

Priority of which item gets shown first can be explicitly defined using the `priority` property of [Subcommands](/docs/autocomplete/api#Subcommand-Object), [Options](/docs/autocomplete/api#Option-Object), and [Suggestions](/docs/autocomplete/api#Suggestion-Object).

## Configuring priority of subcommands
Take an example spec for `git`:

```ts
export const completionSpec: Fig.Spec = {
    name: "git",
    subcommands: [
        {
            name: "config",
            priority: 0
        },
        {
            name: "commit",
            priority: 1
        }
    ]
}
```

In this example, since it's likely that users will be running `git commit` more often than they run `git commit`, it makes sense to give `commit` a higher priority. Now, when a user enters `git c`, commit will be the first subcommand to select.


## Configuring priority of suggestions within generators

Priorities can be used in the suggestion objects created by generators. Take the generator used to stage files in `git add`:

```ts
var files_for_staging = {
    script: "git status --short",
    postProcess: function (out) {
        if (out.startsWith("fatal:")) {
            return [];
        }
        // out = out + " "
        var items = out.split("\n").map(function (file) {
            file = file.trim();
            var arr = file.split(" ");
            return { working: arr[0], file: arr.slice(1).join(" ") };
        });
        return items.map(function (item) {
            var file = item.file;
            var ext = "";
            try {
                ext = file.split(".").slice(-1)[0];
            }
            catch (e) { }
            if (file.endsWith("/")) {
                ext = "folder";
            }
            return {
                name: file,
                insertValue: file.includes(" ") ? "'" + file + "'" : file,
                icon: "fig://icon?type=" + ext + "&color=ff0000&badge=" + item.working,
                description: "Changed file",
                priority: 100,
            };
        });
    },
}
```

Here, the generator runs a script to grab all the unstaged files in a git repo. Then, a high priority is given to all generated objects, since it's most likely that someone will wants to stage these files.