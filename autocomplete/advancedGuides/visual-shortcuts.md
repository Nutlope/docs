# Building Visual Shortcuts with Fig

You can build visual shortcuts with Fig autocomplete to access commonly used commands quickly. It looks like this:

![](/docAssets/autocomplete/visual-shortcuts/menu.png)

Let's build a spec to add a few shortcuts that would level up the Fig development workflow. We want to activate our shortcut menu when we enter `+` into the terminal.

In your local autocomplete/dev folder, create a new file called `+.ts`. This signals to Fig that this spec will be loaded when `+` is detected. In that file, we have the following completion spec:

```ts
export const completionSpec: Fig.Spec = {
  name: "+",
  description: "Fig shortcuts",
  subcommands: [
    {
      icon: "💻",
      name: "Jump to autocomplete repo",
      insertValue: "\b\bcd ~/fig/projects/autocomplete\n",
      description: "Go ~/fig/projects/autocomplete",
    },
    {
      icon: "🛠",
      name: "Start dev server",
      insertValue: "\b\bnpm run dev",
      description: "npm run dev",
    },
    {
      icon: "fig://icon?type=github",
      name: "withfig/autocomplete",
      insertValue: "\b\bopen https://github.com/withfig/autocomplete",
      description: "Open Fig Autocomplete Github page",
    },
    {
      icon: "fig://icon?type=github",
      name: "withfig/fig issues",
      insertValue: "\b\bopen https://github.com/withfig/fig/issues",
      description: "Open Fig Issues Github page",
    },
    {
      icon: "fig://icon?type=git",
      name: "Push to staging",
      insertValue: "\b\bgit push origin staging",
      description: "Push current repo to origin/staging",
    },
    {
      icon: "fig://icon?type=heroku",
      name: "View logs",
      insertValue: "\b\bheroku logs --tail",
      description: "Tail logs for current heroku app",
    },
  ],
};

```

The `insertValue` property is used to insert custom commands into the terminal. Note that each `insertValue` contains two `\b` characters. These are backspace characters that remove the `+ `  at the beginning, so what's entered into the terminal is exactly the content of the `insertValue` property.

Our spec includes a number of convenience methods to make navigating the Fig developer lifecycle easier. The first subcommand inserts a `cd` command with a predefined path into the terminal when it is hit. Other use cases could be setting up shortcuts for deployment, or getting to the right Github pages right from your terminal.