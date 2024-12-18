
1. In /etc/systemd/system/ollama.service.d/environment.conf, set OLLAMA_HOST to this new IP address, in this example it should look something like this,
/etc/systemd/system/ollama.service.d/environment.conf

```bash
[Service]
Environment="OLLAMA_HOST=170.20.138.60:11434"
Environment="OLLAMA_ORIGINS=*"
```

You'll want to restart your ollama service at this point with

```bash
sudo systemctl daemon-reload
sudo systemctl restart ollama
```

    At this point, your ollama service should be pointed at your WSL 2 virtualized ethernet adapter and the next step is to create a port proxy in order to talk to the WSL 2 virtual machine over your network. Open a Powershell window in administrator mode. For reference, serverfault thread

```bash
New-NetFireWallRule -DisplayName 'WSL firewall unlock' -Direction Outbound -LocalPort 11434 -Action Allow -Protocol TCP

New-NetFireWallRule -DisplayName 'WSL firewall unlock' -Direction Inbound -LocalPort 11434 -Action Allow -Protocol TCP
```

and with the WSL firewall rules in place you should be able to run the following to make a port proxy

netsh interface portproxy add v4tov4 listenport=11434 listenaddress=0.0.0.0 connectport=11434 connectaddress=170.20.138.60

and BAM! You should now be able to access the ollama instance on your network!

One caveat I should note, for some weird reason, when I go to http://0.0.0.0:11434 in my machine's browser that's running ollama, I'm not able to connect to the instance, however if I go to my machine's IP, http://192.168.1.123:11434 in the browser, I can access it no problem.

Anyway, hope others find this to be helpful 😁