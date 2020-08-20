---
title: vim
tags: vim
date: 2019-05-08 22:13:36
---

### Multiple file edit
Open multiple files in separate tab via `vim -p file1.txt file2.txt`. 
If already open just use `:tabe file2.txt`.
Just 1gt, 2gt, use `gt` to view.
Close tabs using `:tabc`

### Replace string globally
:%s/release1/release2/g

### evil-tutor
/ followed by a phrase searches FORWARD for the phrase
? followed by a phrase searches BACKWARD for the phrase
After a search type n to find the next occurrence in the same direction
or N to search in the opposite direction.

Type % to find a matching ),], or }
The cursor should be on the matching parenthesis or bracket.

# a way to change errors
:s/old/new to substitute 'new' for first 'old's on a line type
:s/old/new/g to substitute 'new' for all 'old's on a line type
:#,#s/old/new/g to substitute phrases between two line #'s type
:%s/old/new/g to substitute all occurrences in the file type
:%s/old/new/gc to ask for confirmation each time add 'c'

# execute an external command
:! external command
# writing files
:w TEST 
# remove the file
:!rm TEST
# to save part of the file
:#,# w FILENAME
# retrieves disk file FILENAME and inserts it into the current buffer following the cursor position
:r FILENAME
# opens a line BELOW the cursor and places the cursor on the open line in insert state
o
# opens a line ABOVE the cursor and places the cursor on the open line in insert state
O
# R to replace more than on character
R enters Repace mode until <ESC> is pressed to exit.

