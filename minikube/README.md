# Minikube on Windows using VMWare Workstation

## Steps

Download the .exe file for Windows and rename the file to docker-machine-driver-vmware.exe and place it somewhere in your path.
I picked C:\Windows\System32 but anywhere that’s set up in your ENV should be fine.

```bash
curl https://github.com/machine-drivers/docker-machine-driver-vmware/releases/download/v0.1.0/docker-machine-driver-vmware_linux_amd64 -o %windir%\System32\docker-machine-driver-vmware.exe
```

Create the vm and start minikube

```
minikube start --memory=4096 --cpus=4 --vm-driver vmware
```

The VM will be in %USERSPROFILE%\.minikube\machines\minikube

```
user: docker
password: tcuser
```

Troubleshooting

```
minikube start --alsologtostderr -v=7 to debug crashes
```

Reboot, delete your minikube instance and try again

```bash
minikube delete
```

## Documentation

[Install Minikube](https://github.com/kubernetes/minikube)
