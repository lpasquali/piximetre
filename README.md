## piximetre



under Debian GNU/Linux I run piximetre like this:

```sh
docker run -it --rm -v /tmp/.X11-unix:/tmp/.X11-unix -e DISPLAY=unix$DISPLAY -v $HOME/Pictures:/home/semilanceata/Pictures lpasquali/piximetre
```

under OSX High Sierra I use this script:

```sh     
#!/bin/bash
screen -S socat-piximetre -d -m socat TCP-LISTEN:6000,reuseaddr,fork UNIX-CLIENT:\"$DISPLAY\"
ip=$(ifconfig en0 | grep inet | awk '$1=="inet" {print $2}') && echo "My IP is: $ip"
xhost + ${ip}
docker run --rm -it -e DISPLAY=${ip}:0 -v /tmp/.X11-unix:/tmp/.X11-unix --name piximetre lpasquali/piximetre
xhost - ${ip}
screen -S socat-piximetre -X quit
killall -9 socat
```
