## Generate Basics Auth Password

```bash
echo $(htpasswd -nb <USER> <PASSWORD>) | sed -e s/\\$/\\$\\$/g
```

## Grab Text

```bash
cat file.txt |cut -d " " -f2.-f3
```



## Encode Data

```bash
echo -n 'secret_key' | openssl base64
echo p@ssw0rd | base64
echo cEA1NXdvcmQK | base64 --decode
```