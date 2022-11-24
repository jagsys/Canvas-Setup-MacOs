# Canvas-LMS Installation Walkthrough on MacOs (Monterey)

This guide contains detailed instructions for the installation and configuration of [Canvas-LMS](https://github.com/instructure/canvas-lms) on a macOS Monterey system. Also, you can find some information that can be used for troubleshooting.

# Requirements

## Port 80 Availability Check 

Dory uses standard port 80 for HTTP. Before installation is highly recommended to verify that no other processes are listening on port 80.

```bash
lsof -nP -iTCP -sTCP:LISTEN | grep 80 | awk '{print $2}' | xargs -L1 -I{} ps -f {}
```

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

* [Apple chip](https://desktop.docker.com/mac/main/arm64/Docker.dmg?utm_source=docker&utm_medium=webreferral&utm_campaign=dd-smartbutton&utm_location=module)
2. Double-click Docker.dmg to open the installer, then drag the Docker icon to the Applications folder.

3. Double-click Docker. app in the Applications folder to start Docker

4. Accept the agreement window. 

## Configure Docker desktop

1. On Settings -> Resources, assign at least:
  * 8 GB of memory 
  * 150 GB of Disk image size

## [Discontinued] Mutagen

Install
```bash
brew install mutagen-io/mutagen/mutagen mutagen-io/mutagen/mutagen-compose
```
# Canvas-lms

## Instalation

1. Create a code workspace where you’ll place the application repositories
```bash
mkdir ~/workspace
cd ~/workspace
```

2. Pull the repository
```bash
git clone https://github.com/instructure/canvas-lms.git
```

3. Run the script to set up a development environment with Docker automatically.
```bash
cd canvas-lms
./script/docker_dev_setup.sh
```
### During the script execution:
1. Provide your mac's password when asked (one time)
2. Press [y] to copy all the yml files from ./docker-compose/config to ./config 
3. Be patience...
4. Type _DROP_ when asked what to do with the current database
5. Type your email and password for the canvas admin account 
6. Type an organization's name for canvas
7. Choose [1] for the collection of usage data
8. Verify at the end of the script execution, the message "*\o/ Success!*"

## Registering other services in http proxy

Add the following service name's list in docker-compose.yml, inside section: services->web->external_links
* dory_dinghy_http_proxy:google-drive-lti.box
* dory_dinghy_http_proxy:rollcall.docker
* dory_dinghy_http_proxy:canvadocs.docker
* dory_dinghy_http_proxy:office365.docker

```bash
cd ~/workspace/canvas-lms/; ruby -e 'require "yaml"; data = YAML.load_file "docker-compose.yml"; data["services"]["web"]["external_links"] = ["dory_dinghy_http_proxy:google-drive-lti.box","dory_dinghy_http_proxy:rollcall.docker","dory_dinghy_http_proxy:canvadocs.docker","dory_dinghy_http_proxy:office365.docker"];  File.open("docker-compose.yml", "w") { |f| YAML.dump(data, f) }'
```

## Running Canvas for the first time

1. Run
```bash
cd ~/workspace/canvas-lms
docker compouse up
```
2. Wait until all the compilation processes finish (be patience...)

3. Open http://canvas.docker

4. Sign in with your admin credentials

5. Test functionality

6. Log out

7. Shutdown mutagen-compose from another terminal window
```bash
cd ~/workspace/canvas-lms
docker compose down
```

## Running Canvas

**Tip:** to skip webpacker for compiling assets in every run, comment out: "__command: yarn run webpack__" in docker-compose.override.yml 
```bash
sed -i '' 's/command\:\ yarn\ run\ webpack/\#command\:\ yarn\ run\ webpack/' ~/workspace/canvas-lms/docker-compose.override.yml
```
<a name="run_canvas"></a>
1. Run 
```bash
cd ~/workspace/canvas-lms
mutagen-compose up -d
```
<a name="stop_canvas"></a>
2. Stop
```bash
cd ~/workspace/canvas-lms
mutagen-compose down 
```


<a name="dory_install"></a>
# dory

1. Install
```bash
brew install dory
```

2. Download ~/.dory.yml configuration file  
```bash
curl https://raw.githubusercontent.com/jagsys/Canvas-Setup-MacOs/main/dory.yml --output ~/.dory.yml
```

### M1 dory images

On M1 add this images to your services on your ~/.dory.yml or run the following script
* image: tripox/dory-dnsmasq:latest
* image: tripox/dory-http-proxy:latest
```bash
ruby -e 'require "yaml"; data = YAML.load_file ".dory.yml"; data["dory"]["dnsmasq"]["image"] = "tripox/dory-dnsmasq:latest"; data["dory"]["nginx_proxy"]["image"] = "tripox/dory-http-proxy:latest";File.open(".dory.yml", "w") { |f| YAML.dump(data, f) }'
```


3. [Dont Do it !!! We work with dory's proxy ;) ]Disable dory's proxy service (canvas comes with its proxy)
```bash
cd ~; ruby -e 'require "yaml"; data = YAML.load_file ".dory.yml"; data["dory"]["nginx_proxy"]["enabled"] = false;  File.open(".dory.yml", "w") { |f| YAML.dump(data, f) }'
```


4. Change your dns configuration, to use dory as your dns service
```bash
sudo networksetup -setdnsservers Wi-Fi 127.0.0.1 8.8.8.8
```

## Running dory
1. Run
```bash
dory up
```
2. Stop
```bash
dory down
```

Check dory's status
```bash
dory status
```

# Moving from _canvas.docker_ to _canvas.box_

To use some of google's services, canvas virtual host must have a three letters domain suffix. In this case, it's necessary to change our virtual host from canvas.docker to canvas.box ([requires dory](#dory_install))

1. [Stop canvas](#stop_canvas)

2. Replace "canvas.docker" with "canvas.box" in file: ~/workspace/canvas-lms/docker-compose.override.yml 
```bash
sed -i '' 's/\.canvas\.docker/\.canvas\.box/' ~/workspace/canvas-lms/docker-compose.override.yml
```

3. Replace "canvas.docker" with "canvas.box" in file: ~/workspace/canvas-lms/config/domain.yml
```bash
sed -i '' 's/\canvas\.docker/\canvas\.box/' ~/workspace/canvas-lms/config/domain.yml
```
4. [Run canvas](#run_canvas)

5. Open [http://canvas.box](http://canvas.box)


# Troubleshooting

## IP aliases 

* You should have this new IP address alias in you localhost interface [lo0]: *192.168.42.42*
```bash
ifconfig | grep "inet " | grep -Fv 127.0.0.1 | awk '{print $2}'
```

* Verify if you receive a response when pinging to _canvas.box_
```bash
ping canvas.box
```
## Containers

After "mutagen-compose up" command execution, verify if you have these containers running
 * canvas-lms-jobs-1
 * canvas-lms-webpack-1
 * canvas-lms-redis-1
 * canvas-lms-mutagen-1
 * canvas-lms-web-1
 * canvas-lms-postgres-1
 * http-proxy

```bash
cd ~/workspace/canvas-lms
mutagen-compose up -d
sleep 30
docker inspect --format '{{.Name}}' $(docker ps -q)
```
## Unexpected erros while git cloning
This kind of error could be soveld by raising the postBuffer size
```bash
git config --global http.postBuffer 1048576000
```

## Expose docker port
To test locally any service you can export docker ports adding these lines in the service section of your docker-compose.override.yml, :
```bash
    ports:
      - "[Local_Port]:[Docker_Port]"
```

To check if port is open you can run:
```bash
nc -zv 127.0.0.1 [Local_Port]
```
To interact with canvas-lms-web docker
```bash
docker exec -it canvas-lms-web-1 /bin/bash
```
To remove all docker images
```bash
docker stop $(docker ps -a -q)
docker rmi -f $(docker images -aq)
docker volume ls -f dangling=true

```
To remove images without at least one container associated 
```bash
docker image prune -a
```

## Generate SSL Certs for dory 
Dory can only use one cert by time. It must be default.crt To generate a wilcard cert *.box follow this steps in your ~./dory/certs folder

1. Create the CA Private Key (Set the password)
```bash
openssl genrsa -des3 -out CAPrivate.key 2048
```
2. Generate the CA Root certificate (Enter your password, and could use common name = *.box)
```bash
openssl req -x509 -new -nodes -key CAPrivate.key -sha256 -days 3650 -out CAPrivate.box.pem
```
3. Create a Private Key
```bash
openssl genrsa -out default.key 2048
```

4. Generate the CSR (common name = *.box)
```bash
openssl req -new -key default.key -out BoxRequest.csr
```
5. Save this in your box.cnf
```bash
vim box.cnf
```
```
basicConstraints=CA:FALSE
subjectAltName=DNS:*.box
extendedKeyUsage=serverAuth
```
6. Generate your cert
```bash
openssl x509 -req -in BoxRequest.csr -CA CAPrivate.box.pem -CAkey CAPrivate.key -CAcreateserial -extfile box.cnf -out default.crt -days 3650 -sha256
```
7. Open your cert in your keychain and chose "Always trust"
```bash
open default.crt
```
8. Restart dory
```bash
dory restart
```
9. Test Certs
```bash
9 Test
openssl x509 -in default.crt -text -noout
openssl rsa -in default.key -check

openssl x509 -noout -modulus -in default.crt | openssl md5
openssl rsa -noout -modulus -in default.key| openssl md5
```

### Clean DNS Cache
```
sudo dscacheutil -flushcache; sudo killall -HUP mDNSResponder.
```
### Check DNS servers configured
```
networksetup -setdnsservers Wi-Fi
```
