# Building Internal CLIs

Fig allows teams to build autocomplete for their internal CLIs and scripts. A "team" is anyone logged in to Fig with the same email domain as you

To invite your team mates to Fig, get your personal referral link by running `fig invite`

## Use the Autocomplete Boilerplate Template

The boilerplate repo contains a copy of our autocomplete repo, but without the generated specs. This can be used to write and test an internal spec, as well as be the files for an internal CLI hosted on an internal repo.

Head to our boilerplate template repo, and click "Use this template" to make a copy of the repo.
https://github.com/withfig/autocomplete-boilerplate

![](/docAssets/autocomplete/building-internal-clis/usetemplate.png)


## Setting Up The Autocomplete Repo

Clone down your new templated repo to your local machine.

```bash
git clone [git url] [path]
```

When developing a spec, turn on Fig development mode by running `npm run dev` from your autocomplete folder. This should generate a `dev/` folder in the root of your autocomplete folder. 

Development mode compiles all changes from the `dev/` folder into specs that can be read by Fig, and tells Fig to load specs from the `specs/` folder. (If you don't see `specs/` yet, it will be generated upon making changes to any file in `dev/`) This means you can test your internal CLI from your terminal in real time!

Create a TypeScript file in the `dev/` folder for the internal command you're building for.

## Writing Autocomplete For Your Internal CLI

We'll be building a basic internal CLI for Fig. Here are a few commands we want to generate autocomplete suggestions for. (These are actually outward facing commands, but they show the spec format).

`fig invite`

`fig settings [setting subcommand] [argument]`

Here's the completion spec for the `fig invite`.
```ts
export const completionSpec: Fig.Spec = {
    name: "fig",
    description: "Autocomplete for your terminal",
    subcommands: [
        {
            name: "invite",
            description: "share Fig with a teammate ⭐",
            icon: "fig://icon?type=invite",
        },
    }
}
```
The overall spec includes `invite` as a subcommand of `fig`, with a simple description and icon. 

As a more complex example, here's the full spec including the settings subcommand.

```ts
export const completionSpec: Fig.Spec = {
    name: "fig",
    description: "Autocomplete for your terminal",
    subcommands: [
        {
            name: "invite",
            description: "share Fig with a teammate ⭐",
            icon: "fig://icon?type=invite",
        },
        {
            name: "settings",
            description: "fig settings",
            subcommands: [
                {  
                    name: "autocomplete.theme",
                    icon: "fig://icon?type=commandkey",
                    description: "Change Fig's theme",
                    args: {
                        name: "mode",
                        suggestions: [
                            { name: "light", icon: "fig://icon?type=string" },
                            { name: "dark", icon: "fig://icon?type=string" },
                        ],
                    },
                },
            },
        },   
    }
}
```

The settings subcommand has a nested subcommand in it, which takes an argument to change Fig's setting. Following the completion spec format for your internal tool should allow you to build comprehensive autocomplete for any internal tools.

For more information on building more complex autocomplete specs, see [Building Your First Autocomplete Spec](/docs/autocomplete/guides/building-first-spec) and [API](/docs/autocomplete/api).


## Sharing Your Internal CLI with Teammates

Let's say your company has an internal CLI called `acme`. 

You can build a completion spec for your CLI tool and upload it to Fig's cloud.

```bash
fig team:upload </path/to/completion_spec.js>
```

Now you can tell your team members with Fig installed to download the spec you just pushed. Every time an update is pushed, team members can run the same command to pull down the updated specs.

```bash
fig team:download
```

Under the hood, we are saving the spec to Fig's cloud namespaced by your domain name. Then when user's download it, we save it in the `~/.fig/teams/autocomplete` folder and symlink it to their `~/.fig/autocomplete` folder.