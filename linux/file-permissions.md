## Permissions in symbolic notation

`The permissions on files and directories span four scopes:`

| Scope	| Symbol | Description |
| --- | --- | --- |
| User	| u	| The owner of the file or directory |
| Group	| g	| The group of users to who can access the file or directory |
| Other	| o	| Other users (world) |
| All	| a	| All users (world) |
| User	| u	| The owner of the file or directory |
| Group	| g	|The group of users to who can access the file or directory |
| Other	| o	| Other users (world) |
| All	| a	| All users |

## File Permissions

| Permission type | Symbol	| If a file has this permission, you can: |	If a directory has this permission, you can: |
| --- | --- | --- | --- |
| Read	| r	| Open and view file contents (cat, head, tail)	Read directory contents (ls, du) |
| Write	| w	| Edit, delete or rename file (vi)	Edit, delete or rename directory and files within it; create files within it (touch) |
| Execute | x | Execute the file	Enter the directory (cd); without x, the directory’s r and w permissions are useless |
| None	| -	| Do nothing |	Do nothing |

## Permission-Related Commands

| Command | Description |
| --- | --- |
| chmod permission foo	| Change the permissions of a file or directory foo according to a permission in symbolic or octal notation format. Examples: |
| chmod +x foo | Grant execute permissions to all users to foo using symbolic notation. |
| chmod 777 foo | Grant read, write and execute permissions to all users to foo using octal notation. |
| chown user2 foo |	Change the owner of foo to user2. |
| chgrp group2 foo | Change the group to which foo belongs to group2. |
| umask	| Get a four-digit subtrahend. Recall in subtraction: minuend – subtrahend = difference If the minuend is 777, the difference is your default directory permissions; if it’s 666, the difference is your default file permissions. |
| su / sudo / sudo -i | Invoke superuser privileges. |
| id | Find your user id and group id. |
| groups | Find all groups to which you belong. |

`If you run a command beyond the permissions granted, you get errors such as “Permission denied” or “Operation not permitted”.`

## Changing Permissions

`There are two methods to represent permissions on the command line. The first argument of the chmod command admits both representations.`

| Method | Format of permission	| Examples | Non-chmod application |
| --- | --- | --- | --- |
| Symbolic notation	| A short text string consisting of one character of [u/g/o/a], one of the assignment symbols [+/-/=] and at least one of [r/w/x]. If you omit u/g/o/a, the default is a. | u+rg-wxo=rx+x (i.e., a+x) | ls -l and ls -ld command outputs, e.g. -rwxrw-r--x Here, - denotes the absence, not the removal, of a permission. |
| Octal notation | three-digit octal number ranging from 000 to 777	| 774 640 | Computing default permissions with umask |

### Symbolic Notation

`This notation is used in the ls -l and ls -ld command outputs, and it uses a combination of u/g/o/a (denoting the scope ), +/-/=, and r/w/x to change permissions. If you omit u/g/o/a, the default is a.`

`The notation +/-/= refers to granting/removing/setting various permissions.`

`Here are some examples of chmod usage with symbolic notation. You may change more than one permission at a time, joining symbolic notations with a comma (,) as shown in the fourth example below.`

| Command in symbolic notation | Change in user (u) permissions	| Change in group (g) permissions	| Change in world (o) permissions |
| --- | --- | --- | --- |
| chmod +x foo  | ✓ Execute |✓ Execute | ✓ Execute |
|               |           |           |           |
| chmod a=x foo	| ☐ Read   | ☐ Read    | ☐ Read   |
|               | ☐ Write  | ☐ Write   | ☐ Write  |
|               | ✓ Execute | ✓ Execute | ✓ Execute |
|               |           |            |           |
| chmod u-w foo	| ☐ Write |(No change) | (No change) |
|                |            |            |           |   
| chmod u+wx,g-x,o=rx foo |	✓ Write |  ☐ Execute  | ✓ Read |
|                         | ✓ Execute |            | ☐ Write |
|                         |           |            | ✓ Execute |
----------------------------------------------------------------

### Octal Notation

`This notation is a three-digit number, in which each digit represents permissions as the sum of four addends 4, 2, and 1 corresponding to the read (r), write (w) and execute (x) permissions respectively.`

    * The first digit applies to the user (owner) (u).
    * The second digit applies to the group (g).
    * The third digit applies to the world (other users) (o).

| Octal digit | Permission(s) granted | Symbolic |
|--------------|--------------|----------|
| 0	| None | [u/g/o]-rwx |
| 1	| Execute permission only | [u/g/o]=x | 
| 2	| Write permission only | [u/g/o]=w |
| 3	| Write and execute permissions only: 2 + 1 = 3	| [u/g/o]=wx |
| 4	| Read permission only | [u/g/o]=r |
| 5	| Read and execute permissions only: 4 + 1 = 5 | [u/g/o]=rx |
| 6	| Read and write permissions only: 4 + 2 = 6 | [u/g/o]=rw |
| 7	| All permissions: 4 + 2 + 1 = 7 | [u/g/o]=rwx |