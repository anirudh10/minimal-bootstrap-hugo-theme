+++
aliases = []
date = "2021-04-18T09:00:00-04:00"
lastmod = 2021-04-18T13:00:00Z
publishdate = 2021-04-18T13:00:00Z
tags = ["systemd", "devops"]
title = "Systemd"

+++
## UseCase

At work, you might have executed "sudo service some-service start" and your service magically starts.

How does that happen? There are two ways I know - systemd and init.d.

Let's look at systemd.

## The How to

1. Create an id-generator-service.service file that you'll provide to systemd.

       [Unit]
       Description=IDGeneratorService
       
       [Service]
       WorkingDirectory=/opt/codebase/id-generator-service
       ExecStart=/opt/codebase/id-generator-service/scripts/run.sh
       User=ubuntu
       Type=simple
       Restart=on-failure
       RestartSec=10
       
       [Install]
       WantedBy=multi-user.target
2. Create the run.sh file that is needed for id-generator-service.

       #!/bin/bash
       
       APP_HOME="/opt/codebase/id-generator-service"
       cd $APP_HOME
       
       OPTS=" -Xmx9046m -Xms1024m -XX:+UseG1GC -XX:+UnlockExperimentalVMOptions -XX:G1NewSizePercent=40 -XX:G1MaxNewSizePercent=75 -Djava.security.egd=file:/dev/./urandom "
       /usr/bin/java $OPTS -jar $APP_HOME/target/id-generator-service-0.0.1-SNAPSHOT.jar
3. Copy the service file to /etc/systemd/system to create this unit.

       ubuntu@ondemand-instance:~$ sudo cp id-generator-service.service /etc/systemd/system/
4. Enable this unit.

       ubuntu@ondemand-instance:~$ systemctl enable id-generator-service
5. It will say this unit is loaded but inactive and dead.

       ubuntu@ondemand-instance:~$ systemctl list-units --all --type=service --no-pager|grep id-generator-service
       id-generator-service.service             loaded    inactive dead    IDGeneratorService    
6. Start it.

       ubuntu@ondemand-instance:~$ sudo service id-generator-service start