---
title: spacemacs
date: 2021-02-01 20:39:06
tags: 
  - spacemacs
  - emacs
  - editor
---
# Basic
| Key                     | Description                                                                                                                               |
| ---------               | :-----------------------------:                                                                                                           |
| SPC p f                 | search find in the same project                                                                                                           |
| SPC p p                 | open project                                                                                                                              |
| SPC w m                 | max the current windows                                                                                                                   |
| ALT ;                   | uncomment                                                                                                                                 |
| gcc                     | comment out a line                                                                                                                        |
| gc                      | comment out highlighted text                                                                                                              |
| SPC w /                 | vertical                                                                                                                                  |
| SPC w -                 | horizontal                                                                                                                                |
| SPC f e d               | edit spacemacs profile                                                                                                                    |
| SPC f f                 | open file                                                                                                                                 |
| SPC w c                 | close the window                                                                                                                          |
| SPC t l                 | toggle lines                                                                                                                              |
| C-x h                   | select the entire buffer                                                                                                                  |
| C-h m                   | show available commands for the current modes                                                                                             |
| SPC a d                 | Dired mode : 1. Copy file : C 2. Delete the file : D 3. Rename the file : R 4. Create a new directory : + 5. Reload directory listing : g |
| C-b                     | Move back one full screen                                                                                                                 |
| C-f                     | Move forward one full screen                                                                                                              |
| C-d                     | Move forward 1/2 screen                                                                                                                   |
| C-u                     | Move back (upA)1/2 screen                                                                                                                 |
| /sudo::/path/to/file    | Open a file with su/sudo                                                                                                                  |
| r in *spacemacs* buffer | Open from the recent file list                                                                                                            |
| SPC o y                 | Copy to clipboard                                                                                                                         |
| SPC o p                 | Paste from clipboard                                                                                                                      |
| /ssh: :/home/szhou/     | ssh remote edit file                                                                                                                      |
| SPC s c                 | clear the highlight of search                                                                                                             |
| "2p                     | paste the 2nd-last yanked lines                                                                                                           |
| SPC r l                 | resume the last helm session                                                                                                              |

# Movement
| Key       | Description                         |
| --------- | :-----------------------------:     |
| C-u       | up half page                        |
| C-d       | down half page                      |
| d-0       | delete to the beginning of the line |

# Magit 
| Key       | Description                                                            |
| --------- | :-----------------------------:                                        |
| SPC g s   | show Magit status view                                                 |
| j k       | position the cursor on a file                                          |
| TAB       | show and hide the diff for the file                                    |
| s         | stage a file (u to unstage fa file and x to discard changes to a file) |
| cc        | commit                                                                 |
| ,c        | finish the commit message                                              |
| p u       | Push to upstream                                                       |
| F (r) u   | Pull tracked branch and rebase                                         |
|           |                                                                        |
[magit](https://www.saltycrane.com/blog/2018/11/magit-spacemacs-evil-magit-notes/ "magit")

# Org mode
| Key            | Description                     |
| ---------      | :-----------------------------: |
| M-x org-mode   | enable Org mode                 |
| M-shift-RET    | Add a TODO list                 |
| C-c C-t        | Mark as completed               |
| C-c C-x C-b    | toogle checkbox                 |
| C-c C-t        | TODO to DONE                    |
| Alt-Up or Down | Move the item                   |
| C-c C-x C-w    | Delete the item                 |
| Alt-Right      | Demote a headline               |
| Alt-Shit-Left  | To promote a subtree            |
| C-c C-e        | To start the exporter           |
| h o            | Export to a web page and open   |
| t A            | Export as an ADCII text file    |
|                |                                 |

# Dired mode
| Key       | Description                     |
| --------- | :-----------------------------: |
| SPC a d   | enable Dired Mode               |
|           |                                 |

# Helm
| Key       | Description                     |
| --------- | :-----------------------------: |
| SPC r l   | resume the last helm session    |
|           |                                 |
# tumx
| Key                                | Description                                                    |
| ---------                          | :-----------------------------:                                |
| prefix :setw synchronize-panes off | (ctrl + y)stop sending same command for all paines in windows  |
| prefix :setw synchronize-panes on  | (ctrl + u)start sending same command for all paines in windows |
| prefix r                           | reload the tmux configuration                                  |
| prefix d                           | Deatach the session                                            |
|                                    |                                                                |

- 
<https://orgmode.org/manual/index.html>
[keybindings](https://gist.github.com/rnwolf/e09ae9ad6d3ac759767d129d52cab1f1 "spacemacs keybindings")
