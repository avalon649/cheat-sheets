### ssh
```bash
ssh2john id_rsa >> for_john.txt

john for_john.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

### zip

```bash
zip2john 8702.zip > hashes4john.txt

john hashes4john.txt --wordlist=/usr/share/wordlists/rockyou.txt
```