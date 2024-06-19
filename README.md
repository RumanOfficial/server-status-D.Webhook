# Get your Linux server's status on Discord
This is a small but usefull script to get your Linux server's status (start, stop and reboot) on Discord via a webhook.

***⚠️ ONLY TESTED ON UBUNTU 24.04 LTS !*** 


### Here's how it works : 
When your server's start, stop or restart, it triggers a service which also triggers the bash script you'll create and sends a webhook request to Discord.
You can make it to mention a certain discord user, but this is not obliged.
To create a webhook, here's the official [Discord docs](https://support.discord.com/hc/en-us/articles/228383668-Intro-to-Webhooks)

## How do I get it to work ?

### Creating the main script
#### First, you'll have to create a bash script, to handle the server starts, stops and reboot.
To make the script you'll have to create a file in the `/sbin` folder, which will contains the script.

For exemple :
`sudo nano /sbin/server-webhook.sh` --> to create the file and edit it.

Then, place the lines below in the doc : 
```bash
#!/bin/bash

WEBHOOK_URL="https://discord.com/api/webhooks/url_to_your_webhook"
# To get a user mentionned (if wanted)
DISCORDUSER="<@YourDiscordIDhere>"

# Handle server start, stop, and restart
if [ "$1" == "start" ]; then
    PAYLOAD=" { \"content\": \"$DISCORDUSER: The server \`$HOSTNAME\` started.\" }"
elif [ "$1" == "stop" ]; then
    PAYLOAD=" { \"content\": \"$DISCORDUSER: The server \`$HOSTNAME\` stopped.\" }"
elif [ "$1" == "restart" ]; then
    PAYLOAD=" { \"content\": \"$DISCORDUSER: The server \`$HOSTNAME\` is rebooting.\" }"
else
    echo "Usage: $0 {start|stop|restart}"
    exit 1
fi

# Send the webhook
curl -X POST -H 'Content-Type: application/json' -d "$PAYLOAD" "$WEBHOOK_URL"
```
To save the file, you'll have to do this key combination : `CTRL + X`, and then when it request the name, just do `Y`.

Make the script executable by the server : 
``sudo chmod +x /sbin/server-webhook.sh``

### Creating the services
#### Now that the script is done, you'll have to create the services to trigger the script when the server starts, stops or reboot.
We will start at first with the server's reboot service : 

Create a file to contain the service : `sudo nano /etc/systemd/system/webhook-restart.service`

Then place the following lines in it : 
```ini
[Unit]
Description=Send webhook on server restart
Before=shutdown.target reboot.target

[Service]
Type=oneshot
ExecStart=/sbin/server-webhook.sh restart

[Install]
WantedBy=reboot.target

```
As said before, use the keys `CTRL + X` followed by `Y` to save the file.

#### Create the script to handle the server start :
First, create the file which will contain the service : `sudo nano /etc/systemd/system/webhook-start.service`

Then place the following lines inside of it : 
```ini
[Unit]
Description=Send webhook on server start
After=network-online.target
Wants=network-online.target

[Service]
Type=oneshot
ExecStart=/sbin/server-webhook.sh start

[Install]
WantedBy=default.target

```
As always, use `CTRL + X` followed by `Y` to save the file.

#### Finaly, create the service to handle a server stop : 
Create the file to contain the service : `sudo nano /etc/systemd/system/webhook-stop.service`

Then, place the following lines inside : 
```ini
[Unit]
Description=Send webhook on server stop
Before=shutdown.target

[Service]
Type=oneshot
ExecStart=/sbin/server-webhook.sh stop

[Install]
WantedBy=halt.target reboot.target

```
Use `CTRL+X` followed by `Y` to save the file.

### Enable the services (to get them to work)
First, we'll have to get the systemd-deamon to know that these services have been created : `sudo systemctl daemon-reload`

Now, we'll activate them : 
```bash
sudo systemctl enable webhook-start.service
sudo systemctl enable webhook-stop.service
sudo systemctl enable webhook-restart.service
```

### After these, the script should be working ! 
Enjoy !


## Test the script : 
If you want to test the script, you can do this : 
```bash
sudo systemctl start webhook-start.service
sudo systemctl start webhook-restart.service
sudo systemctl start webhook-stop.service
```

## Got any error ? 
If you get any error, you can still ask ChatGPT or me in a issue !
