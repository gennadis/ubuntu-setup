```sh
midclt call system. advanced update '{ "kernel_extra_options": "consoleblank=60" }'
```

```sh
sudo nano /etc/systemd/logind.conf

HandleLidSwitch=ignore
HandleLidSwitchExternalPower=ignore
HandleLidSwitchDocked=ignore
```
