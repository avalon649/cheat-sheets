#                                           Essential Linux Commands

###             1. File and Directory Management
```
*   ls              --      List files and directories
*   cd              --      Change directory
*   pwd             --      Print working directory
*   cp              --      Copy files and directories
*   mv              --      Move or rename files and directories
*   rm              --      Remove files and directories
*   mkdir           --      Create a directory
*   rmdir           --      Remove directory
*   touch           --      Change file timestamp or create empty file
*   find            --      Search for files in a direcory hierachy
*   locat           --      Find files by name
*   tree            --      Display direcories ina tree-like format
*   chmod           --      Change file permissions
*   chown           --      Change file owner and group
*   chgrp           --      Change group ownership
*   stat            --      Display file or file-system status
```
###                2. File Viewing and Editing
```
*   cat             --      Concatenate and display file content
*   tac             --      Concatenate and display file content in reverse
*   more            --      View fike content interactively (page by page)
*   less            --      View fike content interactively (scrollable)
*   head            --      Output the first part of a file
*   tail            --      Output the last part of a file
*   nano            --      Text editor (terminal-based)
*   vim / vi        --      Andvanced text editor
*   emacs           --      Text editor
*   grep            --      Search text using patterns
*   sed             --      Stream editor for filtering and transforming text
*   awk             --      Pattern scanning abd processing language
*   cut             --      Remove sections from each line of files
*   sort            --      Sort lines of text files
```
###                3. Processs Management
```
*   ps              --      Report a snapshot of current processes
*   top             --      Display Linux tasks
*   htop            --      Interactive process viewer (advanced top)
*   kill            --      Send a signal process, typically to terminate
*   killall         --      Terminate process by name
*   pkill           --      Terminate process by name
*   bg              --      Resume a suspended job in the background
*   fg              --      Bring a job in the foreground
*   jobs            --      List active jobs
*   nice            --      Run a programm with modified scheduling priority
*   renice          --      Alter priority of running process
*   uptime          --      Show how long the system has been running
*   time            --      Measure programm running time
```
###                 4. Disk Management
```
*   df              --      Report file system disk usage
*   du              --      Esitmate file system usage
*   fdisk           --      Partition table manipulator for Linux
*   lsblk           --      List information about block device
*   mount           --      Mount a file system
*   umount          --      Unmount a file system
*   parted          --      A partition manipulator programm
*   mkfs            --      Create a file system
*   fsck            --      File system consistency check and repair
*   blkid           --      Locate/print block device attributes
```
###                 4. Networking
```
*   ipconfig        --      Display and configure network interfaces
*   ip              --      Show/manipulate routing, devices, and tunnels
*   ping            --      Send ICMP ECHO_REQUEST packets
*   netstat         --      Network statistics
*   ss              --      Socket statistics (faster then netstat)
*   traceroute      --      Trace the route packets take to a network host
*   nslookup        --      Query internet name servers intercatively
*   dig             --      DNS lookup utility
*   wget            --      Non-interactive newtork downloader
*   curl            --      Transfer data with URLs
*   scp             --      Secure copy between hosts
*   ssh             --      Secure shell for remote login
*   ftp             --      File transfer protocol client
```
###                 4. User and Group Managmenent
```
*   useradd         --      Add a user to the system
*   usermod         --      Modify a user account
*   userdel         --      Remove a user from the system
*   groupadd        --      Add a group to the system
*   groupdel        --      Remove a group from the system
*   passwd          --      Change a user's password
*   chage           --      Change a user's password expiry information
*   whoami          --      Print the current logged-in user
*   who             --      Show who's logged in
*   w               --      Show who's logged in and what they're doing
*   id              --      Prints user and groupp information
*   groups          --      List all groups
```
###                5. System information and Monitoring
```
*   uname           --      Prints system information
*   hostame         --      Show or set the system's hostname
*   uptime          --      How long the system has been running
*   dmesg           --      Boot and system messages
*   free            --      Prints the system's free memory
*   top             --      Prints linux tasks
*   vmstat          --      Report virtuall memory statistics    
```