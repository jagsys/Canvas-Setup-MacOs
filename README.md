# Canvas-Setup-MacOs

# Requirements

## Xcode 

Install
```bash
xcode-select --install
```

Get the latest update
```bash
software update --all --install --force
```

Check Xcode path
```bash 
xcode-select -p 
```
## Brew

Install 

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Update
```bash
brew update
```

Check brew status
```bash
brew doctor
```

## Docker desktop

1. Download Docker Desktop for Mac

* [Intel chip](https://desktop.docker.com/mac/main/amd64/Docker.dmg?utm_source=docker&utm_medium=webreferral&utm_campaign=docs-driven-download-mac-amd64)

2. Double-click Docker.dmg to open the installer, then drag the Docker icon to the Applications folder.

3. Double-click Docker. app in the Applications folder to start Docker

4. Accept the agreement window. 

## Configure Docker desktop

1. On Settings -> Resources, assign at least:
  * 8 GB of memory 
  * 150 GB of Disk image size

## Mutagen

Install
```bash
brew install mutagen-io/mutagen/mutagen mutagen-io/mutagen/mutagen-compose
```
# Canvas-lms

## Instalation

Create a code workspace where you’ll place the application repositories
```bash
mkdir ~/workspace
cd ~/workspace
```

Pull the repository
```bash
git clone https://github.com/instructure/canvas-lms.git
```

Run the script to set up a development environment with Docker automatically.
```bash
cd canvas-lms
./script/docker_dev_setup.sh
```
### During the script execution:
1. Provide your mac's password when asked (one time)
2. Press [y] to copy all the yml files from ./docker-compose/config to ./config (be patience...)
3. Write _DROP_ when asked what to do with the current database
4. Add your email and password for the canvas admin account 
5. Add an organization's name for canvas
6. Choose [1] for the collection of usage data



## Running Canvas for the first time

Run
```bash
mutagen-compose up
```
Wait until all the compilation processes finish (be patience...)

# Trouble shotting 

## IP aliases 

* You should have this new IP address alias in you localhost interface [lo0]: *192.168.42.42*
```bash
ifconfig | grep "inet " | grep -Fv 127.0.0.1 | awk '{print $2}'
```

* Verify if you receive a response when pinging to _canvas.docker_
```bash
ping canvas.docker
```



