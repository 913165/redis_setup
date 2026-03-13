# check ubuntu version before donwloading and installing checkmk
# install checkmk on ubuntu

https://checkmk.com/download

select linux and select row
```
https://checkmk.com/download?platform=cmk&distribution=ubuntu&release=noble&edition=community&version=2.4.0p23
```

# 1. Download Checkmk for Ubuntu or Debian

```
wget https://download.checkmk.com/checkmk/2.4.0p23/check-mk-raw-2.4.0p23_0.noble_amd64.deb
```


# 2. Install the Checkmk package
```
sudo apt install ./check-mk-raw-2.4.0p23_0.noble_amd64.deb
```

Afterwards we can test if the installation was successful by running the omd version command:

```
omd version
```

# 3. Create a Checkmk monitoring site

```
sudo omd create monitoring
sudo omd start monitoring
```

# check ifconfig and http://your_server/monitoring/

