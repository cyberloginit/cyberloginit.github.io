---
layout: post
title: Visual Studio Code Tweaks on Windows 10
---

# Visual Studio Code Tweaks on Windows 10

## Integrated Terminal
### Set Bash on Windows as default terminal environment.
To open the terminal, Use the Ctrl+` keyboard shortcut.
1. open the command palate with `Ctrl+Shift+P`
2. type `open user setting`
3. search `bash` to locate the setting item
4. add `"terminal.integrated.shell.windows": "C:\\Windows\\System32\\bash.exe"` to `WORKSPACE SETTINGS`

## Work on WSL
__DON'T__ modify any file of the WSL from Windows.

However, we can work on files in our Windows file system from either within the WSL or without.
To sum it up:

File Location | Edit in Windows | Edit in WSL
:---: | :---: | :---:
Windows| Yes| Yes
WSL| No| Yes

1. Windows file system does __not__ include `C:\Users\{username}\AppData\Local\Packages\{CanonicalGroupLimited.UbuntuonWindows}\LocalState\rootfs`
2. WSL file system does __not__ include `/mnt/[c, d, ...]`

To make your life easier, put your work directory in Windows file system (e.g. `/mnt/c/Users/{username}/Documents`).

This way, you can enjoy all the goodies from both Bash(Linux) and Windows, and edit your files from either environment.

### Tricks
1. 
```
alias work='cd /mnt/c/Users/{username}/Documents/{work directory}'
```
2. `code .` to open current directory in Visual Studio Code

## WSL Git
not supported yet:(

## References
1. https://stackoverflow.com/questions/42606837/how-to-use-bash-on-windows-from-visual-studio-code-integrated-terminal[](https://stackoverflow.com/questions/42606837/how-to-use-bash-on-windows-from-visual-studio-code-integrated-terminal)
2. https://code.visualstudio.com/docs/editor/integrated-terminal[](https://code.visualstudio.com/docs/editor/integrated-terminal)
3. https://github.com/Microsoft/vscode/issues/29812[](https://github.com/Microsoft/vscode/issues/29812)
