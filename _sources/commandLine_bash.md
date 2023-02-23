(Unix)=
# command line
================================================================================================================================
Here are some basic Unix commands used in Bash. For more detailed instructions on working with bash-based systems, see the hpc-carpentry lessons for [navigating in bash](http://www.hpc-carpentry.org/hpc-shell/02-navigation/index.html) and [file manipulation in bash](http://www.hpc-carpentry.org/hpc-shell/03-files/index.html).

## Basic Unix commands:

|Navigation commands         |                |
|:--------------------- | :------------------ |
| **Get current dirrectory** | **<span style='color:green'> pwd </span>** 
| **Change directory***      | **<span style='color:green'> cd [path] </span>** |
| Go to home directory  | **<span style='color:green'> cd ~/ </span>** |
| Go up one directory  | **<span style='color:green'> cd ../ </span>** |
| **Make new directory** (folder) | **<span style='color:green'> mkdir [path]/[name] </span>** |
| **list files** in current directory | **<span style='color:green'> ls / </span>** |
| **list files** in another directory | **<span style='color:green'> ls [path]/ </span>** |
| List files that contain a specific name or extension (xyz) | <span style='color:green'> ls *xyz or *.xyz </span>(if extension) |

|File commands         |                |
|:--------------------- | :------------------ |
| **Copy file x to y** | **<span style='color:green'> cp [file x] [file y]</span>** 
| **Copy directory** with all files inside | **<span style='color:green'> cp -r [dir x] [dir y] </span>** |
| **Move / Rename file** | **<span style='color:green'> mv [orig path] [new path] </span>** |
| **Delete file** | **<span style='color:green'> rm [file] </span>** |
| **Delete all files with specified extension** xyz | <span style='color:green'> rm *.xyz </span> |
| **View a text file**  | **<span style='color:green'> cat [file] </span>** |
| **Check memmory size of a directory** | **<span style='color:green'> du -h [directory] </span>** |
| **Change permissions for a file** | **<span style='color:green'> chmod g+rxw [file] </span>** |
| **Change permissions for all files** in a directory | **<span style='color:green'> chmod -R g+rxw .</span>** (from directory)|
| note: to change permissions, characters before + are for the type of person (u=user=yourself, g=group, o=others) and characters after the + are for the type of permission (r=read, w=write, x=execute) |