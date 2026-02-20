# docker-flutter-web
Tips to dockerize your flutter dev env. Hot reload and local debug tools enabled via web-server 


## That trick was originally extracted from 
https://medium.com/@andrejtaneski/hacking-flutters-hot-restart-for-web-developing-with-flutter-in-docker-3c8acc9a8cc7

### Steps

1. Navigate to the root of your app
2. Add Dockerfile as described:
 > FROM  ghcr.io/cirruslabs/flutter:stable AS flutter-dev
 > 
 > RUN apt update
 > RUN apt install -y tmux inotify-tools net-tools
 > 
 > RUN mkdir /app/
 > WORKDIR /app/
 > 
 > COPY ./entrypoint.sh ./entrypoint.sh
 > 
 > EXPOSE 8080
 > 
 > RUN chmod +x entrypoint.sh
 > 
 > ENTRYPOINT ["/bin/bash", "./entrypoint.sh"]
3. Add entrypoint.sh file as described:
 > #!/bin/bash
 >
 > flutter pub get
 >
 > \# start a session that runs the flutter development web server # if non Unix add option --web-hostname 0.0.0.0
 > 
 > tmux new-session -d -s flutterSession -n flutterWindow
 > tmux send-keys -t flutterSession:flutterWindow "flutter run -d web-server --web-port 8080" Enter
 >
 > \# start a session that runs a filewatcher and sends the "R" key to the flutter session when files are changet - triggering a hot restart
 > 
 > tmux new-session -d -s watcherSession -n watcherWindow
 > tmux send-keys -t watcherSession:watcherWindow "while inotifywait -e close_write -r lib/; do tmux send-keys -t flutterSession:flutterWindow "R" Enter; done" Enter
 >
 > export TERM=xterm
 >
 > rm -f /tmp/tmuxpipe && mkfifo /tmp/tmuxpipe && tmux pipe-pane -t flutterSession:flutterWindow -o 'cat >> /tmp/tmuxpipe' && cat /tmp/tmuxpipe
4. Add compose.yml file as described:
 > services:
 >   app:
 >     image: flutter-dev
 >     build: .
 >     volumes:
 >       - ./:/app
 >     ports:
 >       - 8080:8080
5. Open terminal, go to your app root, and execute:
 > docker compose up

That will create new container as detailed in Dockerfile, thanks to https://github.com/cirruslabs for his flutter image




   
