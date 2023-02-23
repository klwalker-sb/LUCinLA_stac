(vimCommands)=
# Vim
================================================================================================================================

## Navigating in Vim:

:::{note} If archaic keyboarding is not your thing, you can use the cursor by modifying the configuration in `.vimrc` in your home directory. Just add "set cursorline" `.vimrc`. [See here.](configVim)
::: 

Vim uses good old fashion keyboard navigation
<img align="right" src="/Images/hjkl.jpg" />                                                            
**<span style='color:green'> h </span>** = **Left**   
**<span style='color:green'> l </span>** = **Right**     
**<span style='color:green'> j </span>** = **Up**     
**<span style='color:green'> k </span>** = **Down**  


**<span style='color:green'> Shift+H </span>** puts cursor at the top of the screen.  
**<span style='color:green'> Shift+L </span>**  puts cursor at the bottom of the screen. 

To <span style='background:yellow'> highlight text, </span> hold down **<span style='color:green'> v </span>** while using the **<span style='color:green'> h </span>** or **<span style='color:green'> l </span>** movement keys.  
To <span style='background:yellow'> highlight a whole line,</span> hold down **<span style='color:green'>Shift+v</span>** from the beginning of the line.  
<span style='background:yellow'> to highlight  
    multiple lines  
    up or down, </span>  
    Use **<span style='color:green'>j</span>** or **<span style='color:green'>k</span>** keys at the same time  
    
To **search** for a keyword<xxx>, type **<span style='color:green'>/<xxx><span>**

## Editing in Vim:
The easiest way to edit text in Vim is to **Enter insert mode by typing <span style='color:green'> i </span>**   
Now you can delete using Delete/Backspace and insert new text at the cursor.   
You can **Exit insert mode with <span style='color:green'> Esc </span>**  

If you prefer Command editing, here are some basic commands:
 |Editing commands                 |                 |
 |:------------|:------------|
 |**Copy** ("yank")|**<span style='color:green'> y </span>**|
 |Copy a line|**<span style='color:green'> yy </span>**|
 |Copy a word|**<span style='color:green'> yw </span>**| 
 |Copy to end of line|**<span style='color:green'> y$ </span>**|
 |**Paste**|***<span style='color:green'> p </span>**|
 |**Delete** character|**<span style='color:green'> x </span>**|
 |Delete highlighted|**<span style='color:green'> d </span>**|
 |Delete whole line|**<span style='color:green'> dd </span>**|
 |Delete word|**<span style='color:green'> dw </span>**|
 |Delete to end of line| **<span style='color:green'> D </span>**|
 |Delete to end of file *** | <span style='color:green'> dG </span>**|
 |**Replace** without confirmation | **<span style='color:green'> :%s/[pattern]/[replacement]/g </span>**|
 |**Replace** with confirmation | **<span style='color:green'> :%s/[pattern]/[replacement]/gc </span>** |
 |**Undo** | **<span style='color:green'> u </span>** |
 |**Redo**| **<span style='color:green'> Ctrl+r  </span>** |
 |**Repeat**| **<span style='color:green'> . </span>** |
 
 ## File commands in Vim:
 | File commands                |                 |
 | :------------   |   :------------ |
 | **save** | **<span style='color:green'> :w </span>** |
 | **save and quit** | **<span style='color:green'> :wq </span>** |
 | **save as**<filename> | **<span style='color:green'> :w [filename] </span>** |
 | **quit** without saving | **<span style='color:green'> :q! </span>** |