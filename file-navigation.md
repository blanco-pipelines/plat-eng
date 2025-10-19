# File Navigation

Understanding the Linux file system establishes a strong foundation in Linux. 
The shell serves as a user interface to the operating system, taking commands from the user, interpreting them, and passing them to the kernel for execution.

The shell will be our main tool for working with files. When working with files, its important to remember that Linux does not have a recycling bin.

## Challenge:


### Part 1: Basic navigation

1. Print the current working directory
   ```
    pwd
   ```
3. Navigate to your home folder
   ```
    cd ~
    # cd
    # cd $HOME
   ```
5. Create a folder named "scripts"
   ```
    mkdir scripts
   ```
7. Briefly review the manual page for mkdir. Notice the -p option.
   ```
    man mkdir
    # use /-p enter to search for "-p", similar to the find feature of a browser. You can use "n" for next and "shift+n" for previous.
   ```
9. Create two folders with one command, "code/app"
    ```
    mkdir -p code/app
    ```
11. Create two more folders "~/code/artifacts/files"
    ```
    mkdir -p ~/code/artifacts/files
    ```
13. Navigate between code/app, home, and code/artifacts/files a few times. Notice how the Tab key will autofill paths.


### Part 2: File Manipulation 

12. Navigate to your home folder. Create an empty file named "data.txt" with a command.
    ```
    cd
    touch data.txt
    ```
14. List the contents of your directory with long listing format. Take note of the date, file permissions, file size and ownership.
    ```
    ls -l
    ```
16. echo something to the standard output of the terminal. Append that output to ~/code/artifacts/data.txt.
    ```
    echo "welcome to Linux"
    # use up key, and tab key for file path
    echo "welcome to Linux" >> ~/code/artifacts/data.txt
    ```
17. Echo something different to ~/data.txt
18. Print the contents of both ~/code/artifacts/data.txt and ~/data.txt to the standard output
    ```
    cat ~/code/artifacts/data.txt
    cat ~/data.txt
    ```
19. copy ~/code/artifacts/data.txt into ~/data.txt. Be sure to make a backup, so we do not lose the contents of data.txt. Confirm the backup worked.
    ```
    # using absolute paths
    cp -b ~/code/artifacts/data.txt ~/data.txt

    # using relative paths
    cd ~
    # cp -b code/artifacts/data.txt data.txt

    # List all files (notice a backup file with a ~ or suffix)
    ls -la


    cat data.txt~ # confirm the backup
    cat data.txt # should be same as cat ~/code/artifacts/data.txt
    ```
20. Rename the data.txt~ file to backup.txt.
    ```
    mv data.txt~ backup.txt
    rm data.txt
    ```

20. Delete ~/data.txt
    ```
    mv data.txt~ backup.txt
    rm ~/data.txt
    ```

## Part 3: File System Structure
    
1. Use the less command to read .bashrc and .bash_profile. These files are shell scripts are profiles that are executed when the associated user logs in.
    ```
    less ~/.bash_profile # use q to exit
    less ~/.bashrc
    ```
2. Navigate to the root folder and run ls. Use the ls command on the folders you see. Different operating systems may have different locations for some folders.
   ```
    cd /
    ls
    ls /bin        # contains binaries, or executables
    ls /etc        # contains configurations and directories for dropping configuration files
    ls /etc/logrotate.d # contains configurations for log rotating
    ls /etc/sudoers.d # contains configurations sudo users 
    ls /var        # contains application variable data
    ls /var/log    # contains application logs
    ls /usr        # contains data, executables, files shared across all users
    ls /tmp        # temporary files, cleared on reboot
    ls /root       # home directory for the root user
    ls /home       # home directories for normal users

   ```

## Part 4: Linking Files
19. Run ls -i code/artifacts/data.txt and then ls -i ~/data.txt. The -i is for inode, a unique ID for each file.
    ```
    ls -i ~/code/artifacts/data.txt
    ```
21. Delete data.txt and backup.txt
22. The -i option displays the inode number, a unique ID for each file. Notice that both files share the same inode number — they’re actually the same file. **Remember:** hard links cannot span different file systems.
23. Copy ~/data.txt to ~/backup.txt, but make sure you create a backup (see man page for copy command). Compare the inode numbers.
24. To see the stats of the file, run `stat ~/backup.txt` and `stat ~/data.txt`


### Part 5: File Execution 
14. Navigate to ~/code/artifacts/files. Create a hard link named data.txt here that is linked to ~/data.txt.

21. Run the command below to create a file named ~/code/app/main.py
    ```
    cat << EOF > ~/code/app/main.py
    ```
22.  
