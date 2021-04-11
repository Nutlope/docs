# Icon API

Use this API to get local icons (like file, folder, and application icons) from the macOS device.

The icon API is accessed through the `fig://` URL scheme. 

### File Icons

##### Using Paths
To retrieve the native macOS icon for a path, simply append the path to the file or folder.

```bash
fig:///Users/mschrage/Desktop
```

Fig will expand a leading `~` to the user's `$HOME` directory. So `fig://~/Desktop` is valid and will result an icon equivalent to the URL above. 

> Note: if the specified filepath does not exist on the user's local machine, Fig will fallback to displing an icon based on the file extension. For instance, `fig://~/this-file-does-not-exist.txt` will display the default `.txt` file icon.


##### Using File Extensions

You can also request the icon for a file type directly using the following syntax:
```bash
fig://icon?type=txt
fig://icon?type=png
``` 

To get the generic folder icon, pass in `folder` as the `type`. 
```bash
fig://icon?type=folder
```
### Standard Icons

Fig also bundles a library of icons that can be access through the `fig://` URL scheme. 

`fig://icon?type=NAME` 

The full list as of v1.0.40 is included below:

<img width="16px" height="16px" src="/docAssets/autocomplete/icons/alert.png"> `alert`
<img width="16px" height="16px" src="/docAssets/autocomplete/icons/android.png"> `android`
<img width="16px" height="16px" src="/docAssets/autocomplete/icons/apple.png"> `apple`
<img width="16px" height="16px" src="/docAssets/autocomplete/icons/astrix.png"> `astrix`
<img width="16px" height="16px" src="/docAssets/autocomplete/icons/aws.png"> `aws`
<img width="16px" height="16px" src="/docAssets/autocomplete/icons/azure.png"> `azure`
<img width="16px" height="16px" src="/docAssets/autocomplete/icons/box.png"> `box`
<img width="16px" height="16px" src="/docAssets/autocomplete/icons/carrot.png"> `carrot`
<img width="16px" height="16px" src="/docAssets/autocomplete/icons/characters.png"> `characters`
<img width="16px" height="16px" src="/docAssets/autocomplete/icons/command.png"> `command`
<img width="16px" height="16px" src="/docAssets/autocomplete/icons/commandkey.png"> `commandkey`
<img width="16px" height="16px" src="/docAssets/autocomplete/icons/commit.png"> `commit`
<img width="16px" height="16px" src="/docAssets/autocomplete/icons/database.png"> `database`
<img width="16px" height="16px" src="/docAssets/autocomplete/icons/docker.png"> `docker`
<img width="16px" height="16px" src="/docAssets/autocomplete/icons/firebase.png"> `firebase`
<img width="16px" height="16px" src="/docAssets/autocomplete/icons/gcloud.png"> `gcloud`
<img width="16px" height="16px" src="/docAssets/autocomplete/icons/git.png"> `git`
<img width="16px" height="16px" src="/docAssets/autocomplete/icons/github.png"> `github`
<img width="16px" height="16px" src="/docAssets/autocomplete/icons/gitlab.png"> `gitlab`
<img width="16px" height="16px" src="/docAssets/autocomplete/icons/gradle.png"> `gradle`
<img width="16px" height="16px" src="/docAssets/autocomplete/icons/heroku.png"> `heroku`
<img width="16px" height="16px" src="/docAssets/autocomplete/icons/invite.png"> `invite`
<img width="16px" height="16px" src="/docAssets/autocomplete/icons/kubernetes.png"> `kubernetes`
<img width="16px" height="16px" src="/docAssets/autocomplete/icons/netlify.png"> `netlify`
<img width="16px" height="16px" src="/docAssets/autocomplete/icons/node.png"> `node`
<img width="16px" height="16px" src="/docAssets/autocomplete/icons/npm.png"> `npm`
<img width="16px" height="16px" src="/docAssets/autocomplete/icons/option.png"> `option`
<img width="16px" height="16px" src="/docAssets/autocomplete/icons/package.png"> `package`
<img width="16px" height="16px" src="/docAssets/autocomplete/icons/slack.png"> `slack`
<img width="16px" height="16px" src="/docAssets/autocomplete/icons/string.png"> `string`
<img width="16px" height="16px" src="/docAssets/autocomplete/icons/twitter.png"> `twitter`
<img width="16px" height="16px" src="/docAssets/autocomplete/icons/vercel.png"> `vercel`
<img width="16px" height="16px" src="/docAssets/autocomplete/icons/yarn.png"> `yarn`

If we are missing an icon, please [open an issue](https://github.com/withfig/fig).

### Adding Badges to Icons

You can also add **badges** to icons. Badges are used to provide additional information or to highlight a specific suggestion. 

For instance, the `git` completion spec uses badges to indicate whether files have been renamed, modified or deleted when staging.

You can add a badge to an icon by including `color` or `badge` (or both) as query parameters. 

`color` (optional) must be a 6 character hex color. If not specified, defaults to red (#ff0000).

> Do not include a leading hashtag as the browser will interpret anything following as a URL fragment. 

`badge` (optional) is a unicode string. (You can even use emoji! 🎉)



```bash
fig:///Users/mschrage/Desktop/hello-world.txt?color=2ecc71&badge=✓
fig://icon?type=txt&=2ecc71&badge=✓
```
> Both of these URLs resolve to this image: 
> <img width="16px" height="16px" src="/docAssets/autocomplete/icons/txt-with-check.png">


### Template 🆕

Fig also allows you to generate standalone icons using the same query parameters you use to add badges. 

`color` describes the background color of the icon. If not specified, defaults to white (#ffffff).

`badge` (optional) is a unicode string.

> For example, `fig://template?color=2ecc71&badge=✓` will resolve to this image:
> <img width="16px" height="16px" src="/docAssets/autocomplete/icons/template-check.png">
