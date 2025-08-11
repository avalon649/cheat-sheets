### Base64 Encode

```bash
echo -n 'secret_key' | openssl base64
echo p@ssw0rd | base64
echo cEA1NXdvcmQK | base64 --decode
cat ~/.ssh/your_key | base64 -w 0
```

### Base64 Decode

```bash
echo "password" | base64 -d
```