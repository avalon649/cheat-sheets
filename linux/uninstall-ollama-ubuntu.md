# Uninstalling Ollama on Ubuntu (Official Method)

According to the official Ollama documentation, follow these steps:

## 1. Stop and disable the service

```bash
sudo systemctl stop ollama
sudo systemctl disable ollama
sudo rm /etc/systemd/system/ollama.service
```

## 2. Remove the Ollama libraries

```bash
sudo rm -r /usr/local/lib/ollama
```

## 3. Remove the binary

```bash
sudo rm $(which ollama)
```

## 4. Clean up user, group, and data

```bash
sudo userdel ollama
sudo groupdel ollama
sudo rm -r /usr/share/ollama
```

## 5. Remove user config/models (optional but recommended)

```bash
rm -rf ~/.ollama
```

This last step removes downloaded models and configuration from your home directory, which can be very large (potentially hundreds of GB depending on what models you downloaded).

## Verify removal

```bash
ollama --version
```

If it returns "command not found", the uninstall was successful.

---

**Sources:**
- [Linux - Ollama (Official Documentation)](https://docs.ollama.com/linux)
- [How to Completely Uninstall Ollama - Collabnix](https://collabnix.com/how-to-completely-uninstall-ollama-and-erase-llm-models-on-linux-systems/)
