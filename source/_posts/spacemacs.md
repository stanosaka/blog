---
title: spacemacs
tags: spacemacs
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

# Movement
| Key       | Description                     |
| --------- | :-----------------------------: |
| C-u       | up half page                    |
| C-d       | down half page                  |
|           |                                 |

# Stage files and commit
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


# Org mode
| Key          | Description                     |
| ---------    | :-----------------------------: |
| M-x org-mode | enable Org mode                 |
| M-shift-RET  | Add a TODO list                 |
| C-c C-t      | Mark as completed               |
| C-c C-x C-b  | toogle checkbox                 |
|              |                                 |


# tumx
| Key                                | Description                                                    |
| ---------                          | :-----------------------------:                                |
| prefix :setw synchronize-panes off | (ctrl + y)stop sending same command for all paines in windows  |
| prefix :setw synchronize-panes on  | (ctrl + u)start sending same command for all paines in windows |
| prefix r                           | reload the tmux configuration                                  |
|                                    |                                                                |

- 
<https://orgmode.org/manual/index.html>
