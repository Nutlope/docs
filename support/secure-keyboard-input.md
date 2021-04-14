# Fig and Secure Keyboard Input

Secure Keyboard Input is a feature on MacOS that prevents apps from intercepting key events sent to other apps when typing in private information, like passwords. 

**When Secure Keyboard Input is enabled, Fig cannot provide autocomplete suggestions.**

Apple published a [technical note](https://developer.apple.com/library/archive/technotes/tn2150/_index.html) providing guidance for when it is appropriate to enable Secure Keyboard Input:

> Use `EnableSecureEventInput` only when needed, that is when the keyboard focus moves to a private data entry field. When the keyboard focus changes to a non-secure text field, or when your process detects that it is losing keyboard focus, call `DisableSecureEventInput`.

In most cases, apps will enable it temporarily — when sensitive information is being entered. However, certain apps like iTerm and Terminal allow the user to choose whether to keep it enabled all the time. *Fig will never be able to provide suggestions when this option is selected.*

## Disable in Terminal

![Secure Keyboard Entry in Terminal](/docAssets/support/guide/secure-keyboard-input-terminal.png)

In the menu bar, go to Terminal > Secure Keyboard Entry and make sure that the setting is disabled.

## Disable in iTerm

![Secure Keyboard Entry in iTerm](/docAssets/support/guide/secure-keyboard-input-iterm.png)

In the menu bar, go to iTerm > Secure Keyboard Entry and make sure that the setting is disabled. 

If the checkbox cannot be unselected, iTerm has detected that Secure Input is enabled but it is not the responsible application.  

To figure out how to disable it, continue to the troubleshooting section.

## Troubleshooting

Sometimes other applications enable Secure Input but then fail to turn it off when it is not longer needed.

![Secure Keyboard Entry in Debugger](/docAssets/support/guide/secure-keyboard-input-in-debugger.png)

Clicking the Fig icon in the menubar > Debugger will display the app that is responsible for enabling Secure Keyboard Input.

If this information doesn't appear in the Debugger or the steps above haven't resolved the issue, then run the following command:

```bash
ioreg -l -w 0 |  tr ',' '\n' 2&> /dev/null | grep kCGSSessionSecureInputPID | cut -f 2 -d = | uniq | xargs ps -o command= -p
```


## Secure Input is enabled by Loginwindow Console 

There appears to be a bug with MacOS, where sometimes the login window doesn't terminate (and disable Secure Input) once the user has logged in. ([Source](https://merikan.com/2019/05/troubleshooting-hyperswitch/))

**To fix this, you need to return to the lock screen and then log back in.**

<u>Tip:</u> You can use the `⌃ Control`  `⌘ Command`  `Q` hotkey to quickly go to the login screen.

> If logging in does not resolve the issue, try restarting your machine. 

## Another app is responsible

If the app that shows up isn't a terminal or the loginwindow, TextExpander has helpfully complied a [list of apps](https://textexpander.com/secure-input) that trigger Secure Keyboard Input and how to resolve the issue.  