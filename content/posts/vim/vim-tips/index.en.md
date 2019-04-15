---
title: Vim Tips and Tricks No. 1
author: Avery Wagar
date: 2018-02-02T13:35:00+01:00
draft: false
toc: false
---

Welcome back my fellow Vim enthusiasts! Today we will be discussing a few shortcuts you can add to your __.vimrc__!
## Find and Replace
Refactoring code is a large part of any programmers daily routine. Sometime you have to refactor a variable across a whole file. 
If you add the following code to your __.vimrc__

```vim
" Find and Replace
map <leader>fr :%s///g<left><left> " Find and replace
map <leader>frl :s///g<left><left> " Find and replace (current line only)
```
You can refactor code with a breeze. Just search in Vim using `/` and then hit <leader>fr to start refactoring. Then type in the new word and hit enter. Boom. Your code is refactored.
__Note:__ Currently there is no good way to refactor across multiple files. (Let me know in the comments if you find a way!)

# Clearing searches
Another issue I have with Vim out of the box is searching cannot be cleared easily. So instead of typing a random string into your search bar, you can clear it with the following mapping:
```vim
" Clear searchs
map <leader><space> :let @/=''<cr> " clear search
```


# Switching buffers
While working across multiple files I like to use buffers. I added shortcuts to switch my buffers easily. 


```vim
" Switching Buffers
noremap <leader>[ :bp<return>
noremap <leader>] :bn<return>
```
Now you can page through your buffers with ease.

# Vim hard mode
So if you are new to Vim, or are still used to the 'Arrow keys.' You can enable the little known hard mode in Vim.

```vim
" No arrow keys (Vim Hard mode)
noremap <Up> <NOP>
noremap <Down> <NOP>
noremap <Left> <NOP>
noremap <Right> <NOP>
```
Now you can ascend to the Master Vimmer you are meant to be.

# Conclusion
So now that you have some new tools in your Vim toolbox. I hope you can extend your love for Vim. Be sure to check back soon for part 2! Let me know any other tips I should know in the comments.


