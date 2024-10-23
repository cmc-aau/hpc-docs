
By default, VSCode wants to help you write code. So much so that it starts to get intrusive. When you type, the IntelliSense kicks in and immediately suggests code for you upon each keystroke. When you hover over your code, a popup definition appears. When you type an open parenthesis, it pops up another autocomplete suggestion window. When you type out a tag, a closing tag magically appears, but sometimes it's wrong. I get what VSCode is trying to do but it got to a point where it was annoying me and getting in the way.

If you want VSCode to become more of a passive editor; if you enjoy typing the majority or all of your code, then add these to your settings.json to disable these autocomplete suggestions and pop ups:

```
"editor.autoClosingBrackets": "never",
"editor.suggestOnTriggerCharacters": false,
"editor.quickSuggestions": false,
"editor.hover.enabled": false,
"editor.parameterHints.enabled": false,
"editor.suggest.snippetsPreventQuickSuggestions": false,
"html.suggest.html5": false
```

Experiencing problems with vscode, hanging on remote connection etc. Log in on the particular login node and kill all vscode-server processes:
```
ps ux | grep [.]vscode-server | awk '{print $2}' | xargs kill
```

then try again.