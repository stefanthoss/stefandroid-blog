ARM board from manufacturs like Orange Pi, Pine64, or Odroid.

This is a bit hacky due to the MongoDB dependencies. There might be a better way to make this work.

If you try to install the UniFi controller as described in my [guide for Debian]({% post_url 2021-05-28-unifi-controller-on-debian.md %}), you'll run into error messages because the UniFi and MongoDB repositories don't contain packages for ARM processors.

```text
N: Skipping acquire of configured file 'main/binary-armhf/Packages' as repository 'http://repo.mongodb.org/apt/debian stretch/mongodb-org/3.6 InRelease' doesn't support architecture 'armhf'
N: Skipping acquire of configured file 'main/binary-arm64/Packages' as repository 'http://repo.mongodb.org/apt/debian stretch/mongodb-org/3.6 InRelease' doesn't support architecture 'arm64'
N: Skipping acquire of configured file 'ubiquiti/binary-arm64/Packages' as repository 'https://www.ui.com/downloads/unifi/debian stable InRelease' doesn't support architecture 'arm64'
```

Ubiquiti provides ARM-compatible packages for UniFi, as does AdoptOpenJDK. The MongoDB repository is a bit more complicated because there are no official MongoDB 3.x packages for ARM Debian. There is however one for Ubuntu 16.04 (Xenial) so I'm using that. Unfortunately there is a missing `libssl1.0.0` issue that still needs to be resolved:

```
The following packages have unmet dependencies:                                                                                                                        
 mongodb-org-server : Depends: libssl1.0.0 (>= 1.0.2~beta3) but it is not installable
```

I couldn't find a good way to resolve this dependency so let's just download the deb file from the Ubuntu servers:

```shell
wget -P /tmp/ http://ports.ubuntu.com/pool/main/o/openssl1.0/libssl1.0.0_1.0.2n-1ubuntu5_arm64.deb
sudo apt install -f /tmp/libssl1.0.0_1.0.2n-1ubuntu5_arm64.deb
```

Now I'm to install the UniFi Network Controller, MongoDB, and OpenJDK 8. At the end enable autostart for all services:

```shell
sudo apt update
sudo apt install apt-transport-https ca-certificates gnupg haveged

wget -qO - https://dl.ui.com/unifi/unifi-repo.gpg | sudo apt-key add -
wget -qO - https://adoptopenjdk.jfrog.io/adoptopenjdk/api/gpg/key/public | sudo apt-key add -
wget -qO - https://www.mongodb.org/static/pgp/server-3.6.asc | sudo apt-key add -

echo "deb [arch=armhf] https://www.ui.com/downloads/unifi/debian stable ubiquiti" | sudo tee /etc/apt/sources.list.d/100-ubnt-unifi.list
echo "deb [arch=arm64] https://adoptopenjdk.jfrog.io/adoptopenjdk/deb buster main" | sudo tee /etc/apt/sources.list.d/adoptopenjdk.list
echo "deb [arch=arm64] https://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.6 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.6.list

sudo apt update
sudo apt install unifi adoptopenjdk-8-hotspot mongodb-org-server

sudo systemctl enable unifi.service
sudo systemctl enable mongod.service
sudo systemctl enable haveged.service
```

The UniFi Network Controller UI is now accessible at `https://<YOUR_HOST_IP>:8443/` (if not, check with `sudo systemctl status unifi.service` whether the UniFi service started successfully).
