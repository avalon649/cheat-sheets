### [WSL 2] Exposing ollama via 0.0.0.0 on local network 

1. Get the IP of the WSL 2 virtualized ethernet adapter which can be done by running ifconfig in WSL 2 and getting the IP from the eth0 field, it should be under inet,

```bash
ifconfig

eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 170.20.138.60
```        

in this case, the IP address we'll be using is 170.20.138.60

2. In /etc/systemd/system/ollama.service.d/environment.conf, set OLLAMA_HOST to this new IP address, in this example it should look something like this,
`/etc/systemd/system/ollama.service.d/environment.conf`

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

3. At this point, your ollama service should be pointed at your WSL 2 virtualized ethernet adapter and the next step is to create a port proxy in order to talk to the WSL 2 virtual machine over your network. Open a Powershell window in administrator mode. For reference, serverfault thread

### Adding Firewall Rule
```bash
New-NetFireWallRule -DisplayName 'Ollama' -Direction Outbound -LocalPort 11434 -Action Allow -Protocol TCP

New-NetFireWallRule -DisplayName 'Ollama' -Direction Inbound -LocalPort 11434 -Action Allow -Protocol TCP

```

and with the WSL firewall rules in place you should be able to run the following to make a port proxy

### Removing Firewall Rule

```bash
Remove-NetFireWallRule -DisplayName 'Ollama' -Direction Outbound -LocalPort 11434 -Action Allow -Protocol TCP

Remove-NetFireWallRule -DisplayName 'Ollama' -Direction Inbound -LocalPort 11434 -Action Allow -Protocol TCP
```



```bash
netsh interface portproxy add v4tov4 listenport=11434 listenaddress=0.0.0.0 connectport=11434 connectaddress=170.20.138.60
```