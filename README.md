# CrowdControl

### CrowdControl node Installation Instructions.

[Official documentation](https://github.com/DecentralCardGame/Testnet)

System requirements:</br>
CPU: Quad Core or larger AMD or Intel (amd64) CPU
RAM:32GB RAM
SSD:1TB NVMe Storage
100MBps bidirectional internet connection
OS: Ubuntu 20.04 or 22.04</br>

You can take a weaker server

### Network configurations: </br>
Outbound - allow all traffic </br>
Inbound - open the following ports :</br>
1317 - REST </br>
26657 - TENDERMINT_RP </br>
26656 - Cosmos </br>
Running as a Provider? Add these specific ports: </br>
22221 - provider port </br>
22231 - provider port </br>
22241 - provider port </br>

### Installing the Babylon Node

1. Preparing the server/Required packages installation</br>
```
sudo apt update
sudo apt upgrade -y
sudo apt-get install libclang-dev
sudo apt install -y unzip logrotate git jq sed wget curl coreutils systemd
```
### Go installation.
```
sudo rm -rf /usr/local/go
curl -L https://go.dev/dl/go1.21.6.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile
source .bash_profile
go version
```

### Download and build binaries
```
cd && rm -rf Cardchain
git clone https://github.com/DecentralCardGame/Cardchain
cd Cardchain
git checkout v0.14.2
make install
```

# Config and init app
```
Cardchaind config chain-id cardtestnet-10
Cardchaind config keyring-backend test
Cardchaind config node tcp://localhost:26657
Cardchaind init "your moniker" --chain-id cardtestnet-10
```

# Download genesis and addrbook
```
curl -L https://snapshots-testnet.nodejumper.io/cardchain-testnet/genesis.json > $HOME/.cardchaind/config/genesis.json
curl -L https://snapshots-testnet.nodejumper.io/cardchain-testnet/addrbook.json > $HOME/.cardchaind/config/addrbook.json
```

# Set seeds and peers
```
sed -i -e 's|^seeds *=.*|seeds = "2aa407243c982ce2d9ee607b15418cf45b5002f7@202.61.225.157:20056,947aa14a9e6722df948d46b9e3ff24dd72920257@cardchain-testnet-seed.itrocket.net:31656"|' $HOME/.cardchaind/config/config.toml
sed -i -e 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.01ubpf"|' $HOME/.cardchaind/config/app.toml
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "17"|' \
  $HOME/.cardchaind/config/app.toml
```

# Create service file
```
sudo tee /etc/systemd/system/Cardchaind.service > /dev/null << EOF
[Unit]
Description=CrowdControl node service
After=network-online.target
[Service]
User=$USER
ExecStart=$(which Cardchaind) start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable Cardchaind.service
```

# Reset and download snapshot
```
curl "https://snapshots-testnet.nodejumper.io/cardchain-testnet/cardchain-testnet_latest.tar.lz4" | lz4 -dc - | tar -xf - -C "$HOME/.cardchaind"
```

# enable and start service
```
sudo systemctl start Cardchaind.service
sudo journalctl -u Cardchaind.service -f --no-hostname -o cat
```

### Becoming a Validator

# Create wallet key new
```
Cardchaind keys add wallet
```

(OPTIONAL) RECOVER EXISTING KEY
```
Cardchaind keys add wallet --recover
```

# check sync status, once your node is fully synced, the output from above will print "false"
```
Cardchaind status 2>&1 | jq -r '.SyncInfo.catching_up // .sync_info.catching_up'
```

### We receive tokens in the [site](https://crowdcontrol.network/#/)

# before creating a validator, you need to fund your wallet and check balance
```
Cardchaind q bank balances $(Cardchaind keys show wallet -a)
```
# Create validator
```
Cardchaind tx staking create-validator \
--amount=1000000ubpf \
--pubkey=$(Cardchaind tendermint show-validator) \
--moniker="Moniker" \
--identity=FFB0AA51A2DF5955 \
--details="I love YTWO❤️" \
--chain-id=cardtestnet-10 \
--commission-rate=0.10 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.01 \
--min-self-delegation=1 \
--from=wallet \
--gas-prices=0.01ubpf \
--gas-adjustment=1.5 \
--gas=auto \
-y 
```

### Update
```
No update

Current network:cardtestnet-10
Current version:v0.14.2
```

### Useful commands

Check balance
```
Cardchaind q bank balances $(Cardchaind keys show wallet -a)
```

CHECK SERVICE LOGS
```
sudo journalctl -u Cardchaind -f --no-hostname -o cat
```

RESTART SERVICE
```
sudo systemctl restart Cardchaind
```

GET VALIDATOR INFO
```
Cardchaind status 2>&1 | jq -r '.ValidatorInfo // .validator_info'
```

DELEGATE TOKENS TO YOURSELF
```
evmosd tx staking delegate $(evmosd keys show wallet --bech val -a) 1000000aevmos --from wallet --chain-id zgtendermint_9000-1 --gas-prices 250000000aevmos --gas-adjustment 1.5 --gas auto -y 
```

REMOVE NODE
Please, before proceeding with the next step! All chain data will be lost! Make sure you have backed up your priv_validator_key.json!
```
sudo systemctl stop Cardchaind && sudo systemctl disable Cardchaind && sudo rm /etc/systemd/system/Cardchaind.service && sudo systemctl daemon-reload && rm -rf $HOME/.cardchaind && rm -rf Cardchain && sudo rm -rf $(which Cardchaind)
```
