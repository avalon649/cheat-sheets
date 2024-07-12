### Using Chmod Command to Change File Permissions 

As all Linux users, you will at some point need to modify the permission settings of a file/directory. The command that executes such tasks is the chmod command.

#### The basic syntax is:

chmod [permission] [file_name]

#### There are two ways to define permission:

*  using symbols (alphanumerical characters)
*  using the octal notation method

#### Define File Permission with Symbolic Mode

To specify permission settings using alphanumerical characters, you’ll need to define accessibility for the user/owner (u), group (g), and others (o).

Type the initial letter for each class, followed by the equal sign (=) and the first letter of the read (r), write (w) and/or execute (x) privileges.

To set a file, so it is public for reading, writing, and executing, the command is:

chmod u=rwx,g=rwx,o=rwx [file_name]

To set permission as in the previously mentioned test.txt to be:
* read and write for the user
* read for the members of the group
* read for other users

#### Use the following command:

chmod u=rw,g=r,o=r test.txt

Note: There is no space between the categories; we only use commas to separate them.
Define File Permission in Octal/Numeric Mode

Another way to specify permission is by using the octal/numeric format. This option is faster, as it requires less typing, although it is not as straightforward as the previous method.

#### Instead of letters, the octal format represents privileges with numbers:

    r(ead) has the value of 4
    w(rite) has the value of 2
    (e)x(ecute) has the value of 1
    no permission has the value of 0

#### The privileges are summed up and depicted by one number. Therefore, the possibilities are:

    7 – for read, write, and execute permission
    6 – for read and write privileges
    5 – for read and execute privileges
    4 – for read privileges

As you have to define permission for each category (user, group, owner), the command will include three (3) numbers (each representing the summation of privileges).

For instance, let’s look at the test.txt file that we symbolically configured with the chmod u=rw,g=r,o=r test.txtcommand.

#### The same permission settings can be defined using the octal format with the command:

chmod 644 test.txt