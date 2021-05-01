# Customising Suggestion Objects

You can customize suggestions. View the full specification for a **Suggestion Object** here:  [https://withfig.com/docs/autocomplete/api](https://withfig.com/docs/autocomplete/api#suggestion-object)



![Suggestion Object](https://withfig.com/docAssets/autocomplete/api/suggestionObjects.png)



Some examples:

```javascript
{
	name: "install", // the token used for Fig's parser. Will also be text displayed if not overriden by displayName
	
	displayName: "Install an NPM package 📦", // customise the text displayed for a given suggestion
	
	insertValue: "install ", // change what is inserted when the users selects something
	
	description: "Install a package"
	
	icon: "😃",
	
}
```





