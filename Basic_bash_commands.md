## Description

These are some basic bash commands that you need to know for a quick start on a Linux/Unix system. 

If you use Mac, your can simply search for the **Terminal** app and open it. If you use Windows, you can install Linux through VirtualBox on your Windows. 

## 1. Navigation

The folder we normally referred to is called directory in computing. A directory that is inside another directory is called a subdirectory. The followings are done in your Terminal window. Everything after the hash sign (#) is a comment. So it does not matter if it is included when you copy and paste the commands.

#### Check which directory that you are in: `pwd` *print working directory* 

```bash
pwd # Similar Output: /Users/xwu35. Yours should be /Users/your_computer_username
```

#### Check what files are in your working directory: `ls` *list directory contents*

```bash
ls # Output: Applications Desktop Documents Downloads Library Movies Music Pictures 

ls -l # list all the files with more detail

ls -a # list all the files including the hidden ones
```

#### Enter a directory: `cd` *change directory*

```bash
cd Desktop # change to Desktop directory
```

#### Create a directory: `mkdir` *make directory*

```bash
mkdir test # create a directory called test. You should see this test folder on your desktop now if you did type 'cd Desktop' above
cd test # enter the test directory
```

#### Change to the parent directory of the current one: `cd ..` *the two dots mean above directory*
```bash
cd .. # now you should be back to Desktop directory, use 'pwd' to check

cd ../../ # you will go back two directories above. If you typed this instead of 'cd ..' when you are in test directory, then you should be back to /Users/your_computer_username

cd # if you just type 'cd', then you will go back to your home directory /Users/your_computer_username. this is the same as 'cd ~'
```

## 2. Files

#### Create a file: `cat` *concatenate* or `touch`
```bash
cd test # first we change to test directory
mkdir fun # create another directory called fun inside the test directory. We will use this one later.

cat > my_file.txt # after you execute this command, type 'this is the first line of my file' as the content, then press control + d to stop

# another option to create file using touch and then write into the file using echo
touch my_second_file.txt
# echo: print text or string data into terminal or anothe command
echo "this is the first line of my second file" > my_second_file.txt 
```
#### View file contents: `cat` or `less`
```bash
cat my_file.txt # Output: this is the first line of my file 

# 'cat' prints all the contents on terminal screen. It is not nice to see if it is a large file. If you want to view a large file, you can use 'less' to view the file.
less my_file.txt # press q to quit viewing 
```

#### Copy file: `cp` *copy*; copy directory: `cp -r`
```bash
cp my_file.txt this_is_a_copy.txt # now you copied your file to a new file called 'this_is_a_copy.txt'
cat this_is_a_copy.txt # view the contents of this new file, you should see the same output as my_file.txt. 

cp -r fun copy_fun # copy fun directory. You can use the `ls` command to list all the files: copy_fun  fun   my_file.txt  this_is_a_copy.txt
```

#### Remove file: `rm` *remove* ; remove empty directory: `rmdir`; remove directory that contains files `rm -r`; forcefully remove directory that contains files `rm -rf` 

***Note: DO NOT use `rm -rf` if you don’t know exactly what you are doing.***
```bash
rm this_is_a_copy.txt # remove the file "this_is_a_copy.txt"

rmdir copy_fun # remove the empty directory copy_fun
```
  
- Move file: `mv` *move*
```bash
mv my_file.txt ../Desktop # move my_file.txt to Desktop

pwd # execute this and remember the output path. It should be similar to this: /Users/xwu35/Desktop/test
```

## 3. Path

A path is like a direction to tell the system where to go. Giving a full path enables us to access the file no matter which directory you are in.

Open a new terminal, and type:
```bash
cd /Users/your_username/Desktop/test # use the path that you got from above. Now you are in the test directory
cp -r /Users/your_username/Desktop/test/fun /Users/your_username/Desktop # copy the fun directory to your Desktop
```

## 4. Few tricks

#### DO NOT use space in filenames

Try to avoid using space in your filenames, otherwise it can be quite annoying when you want to use them in command line. You can use capital letters or underscores to separate words in filenames, e.g. testDirectory or test_directory

#### TAB completion

Always use tab to automatically fill in partially typed commands! Say for instance, if you open your terminal and type 'le', then press tab twice, it will give you all the commands starting with letters 'le'. This also works for paths and filenames. If you want to go to /Desktop/test, then type 'cd D' followed by tab(s) will list all the possible directories. 

#### Up and down arrow

up arrow: previous command; down arrow: next command

#### Continuing lines: \
```bash
echo This \
IS \
A \
Long \
Line
```

#### Wildcards

The asterisk * is a wildcard symbol in unix. You can use it for files that have common sections of their name. 

Copy all the txt files from the fun directory to your Desktop
```bash
cp /Users/your_username/Desktop/test/fun/*.txt  /Users/your_username/Desktop/ 
```
***NOTE: DO NOT use wildcards to remove files if you don’t know exactly what you are doing.***

#### Kill a process: Control + c

If you are running a program in your terminal, but later you wish to stop it, then you can hold control and press c to kill the process. For instance, you used cat to view a very large file in your terminal, but you realize that it is too large to view using cat, then you can use control + c to stop the printing on terminal screen.