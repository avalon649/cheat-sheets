```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'

/usr/bin/script -qc /bin/bash /dev/null

stty raw -echo; fg; reset

---- or reset

export TERM=xterm
```