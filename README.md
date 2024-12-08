Make the updates and install the necessary packages.

```
sudo apt update && sudo apt upgrade
```
```
cd $HOME
VER="1.23.0"
wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm "go$VER.linux-amd64.tar.gz"
[ ! -f ~/.bash_profile ] && touch ~/.bash_profile
echo "export PATH=$PATH:/usr/local/go/bin:~/go/bin" >> ~/.bash_profile
source $HOME/.bash_profile
[ ! -d ~/go/bin ] && mkdir -p ~/go/bin
```


install tenderduty

```
sudo addgroup --system tenderduty 
sudo adduser --ingroup tenderduty --system --home /var/lib/tenderduty tenderduty
sudo -su tenderduty
cd ~
echo 'export PATH=$PATH:~/go/bin' >> .bashrc
. .bashrc
git clone https://github.com/blockpane/tenderduty
cd tenderduty
go install
cp example-config.yml ../config.yml
cd
```

Open the Telegram App: Launch the Telegram application on your mobile device or desktop computer.

Find BotFather: Type "BotFather" in the search bar to locate Telegram's official bot creation service.

Start a Chat with BotFather: Navigate to the BotFather profile and start a conversation.

Create a New Bot: Use the /newbot command to initiate the process of creating a new bot. BotFather will ask you a series of questions following this command.


Follow these steps for the CHANNEL section:

Add the Telegram BOT to the group.

Get the list of updates for your BOT:

```
https://api.telegram.org/bot<YourBOTToken>/getUpdates
```
Ex:

```
https://api.telegram.org/bot123456789:jbd78sadvbdy63d37gda37bd8/getUpdates
```

Look for the "chat" object:

```
{
    "update_id": 8393,
    "message": {
        "message_id": 3,
        "from": {
            "id": 7474,
            "first_name": "AAA"
        },
        "chat": {
            "id": <group_ID>,
            "title": "<Group name>"
        },
        "date": 25497,
        "new_chat_participant": {
            "id": 71, 
            "first_name": "NAME",
            "username": "YOUR_BOT_NAME"
        }
    }
}
```
### This is a sample of the response when you add your BOT into a group.

### Add the group ID to the CHANNEL section.

### Fill with your info

```
API_KEY=6549793165xx
CHANNEL=-100xxx
CHAIN_NAME=Dymension
CHAIN_ID=froopyland_100-1
VALOPER_ADDRESS=dymvaloper1xxxx
```
### Setting config.yml (one command, dont touch anything)

```
tee <<EOF >/dev/null config.yml
---

enable_dashboard: yes
listen_port: 8888
hide_logs: yes
node_down_alert_minutes: 3
prometheus_enabled: yes
prometheus_listen_port: 28686

pagerduty:
  enabled: no
  api_key: aaaaaaaaaaaabbbbbbbbbbbbbcccccccccccc
  default_severity: alert
discord:
  enabled: no
  webhook: https://discord.com/api/webhooks/999999999999999999/zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzz
telegram:
  enabled: yes
  api_key: '${API_KEY}'
  channel: "${CHANNEL}"

chains:
  "${CHAIN_NAME}":
    chain_id: ${CHAIN_ID}
    valoper_address: ${VALOPER_ADDRESS}
    public_fallback: yes
    alerts:
      # If the chain stops seeing new blocks, should an alert be sent?
      stalled_enabled: yes
      # How long a halted chain takes in minutes to generate an alarm
      stalled_minutes: 10
      # Most basic alarm, you just missed x blocks ... would you like to know?
      consecutive_enabled: yes
      # How many missed blocks should trigger a notification?
      consecutive_missed: 1
      # NOT USED: future hint for pagerduty's routing
      consecutive_priority: critical
      # For each chain there is a specific window of blocks and a percentage of missed blocks that will result in
      # a downtime jail infraction. Should an alert be sent if a certain percentage of this window is exceeded?
      percentage_enabled: yes
      # What percentage should trigger the alert
      percentage_missed: 10
      # Not used yet, pagerduty routing hint
      percentage_priority: warning
      # Should an alert be sent if the validator is not in the active set ie, jailed,
      # tombstoned, unbonding?
      alert_if_inactive: yes
      # Should an alert be sent if no RPC servers are responding? (Note this alarm is instantaneous with no delay)
      alert_if_no_servers: yes

      # Chain specific setting for pagerduty
      pagerduty:
        enabled: no
        api_key: "" # uses default if blank
      # Discord settings
      discord:
        enabled: no
        webhook: "" # uses default if blank
      # Telegram settings
      telegram:
        enabled: yes
        api_key: "${API_KEY}" # uses default if blank
        channel: "${CHANNEL}" # uses default if blank

       nodes:
      - url: http://176.9.48.38:26657
        alert_if_down: no
      - url: http://135.181.73.170:25157
        alert_if_down: no
      - url: http://5.161.57.83:26557
        alert_if_down: no
      - url: http://89.116.31.118:26657
        alert_if_down: no  

EOF
```

### Exit from tenderduty user

```
exit
```

Create and enable the service (one command)

```
sudo tee /etc/systemd/system/tenderduty.service << EOF
[Unit]
Description=Tenderduty
After=network.target
ConditionPathExists=/var/lib/tenderduty/go/bin/tenderduty

[Service]
Type=simple
Restart=always
RestartSec=5
TimeoutSec=180

User=tenderduty
WorkingDirectory=/var/lib/tenderduty
ExecStart=/var/lib/tenderduty/go/bin/tenderduty

# there may be a large number of network connections if a lot of chains
LimitNOFILE=infinity

# extra process isolation
NoNewPrivileges=true
ProtectSystem=strict
RestrictSUIDSGID=true
LockPersonality=true
PrivateUsers=true
PrivateDevices=true
PrivateTmp=true

[Install]
WantedBy=multi-user.target
EOF
```

### Start systemd service

```
sudo systemctl daemon-reload
sudo systemctl enable tenderduty
sudo systemctl start tenderduty
```

### Check logs 
sudo journalctl -fu tenderduty
