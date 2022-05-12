# Archway Incentivized Testnet (`torii-1`)
### ⚠️ Instructions for Testnet available [here](https://philabs.notion.site/Validator-Setup-Guide-10502472842e4ad8bf7fb7ec68afe07a)

## Hardware Requirements
* **Minimum**
  * 4 GB RAM
  * 200 GB SSD
  * 2 vCPU
* **Recommended**
  * 16 GB RAM
  * 500 GB SSD
  * 6 vCPU

## Installation Steps
### Install `go`
```bash
cd $HOME
wget -O go1.18.1.linux-amd64.tar.gz https://golang.org/dl/go1.18.1.linux-amd64.tar.gz
rm -rf /usr/local/go && tar -C /usr/local -xzf go1.18.1.linux-amd64.tar.gz && rm go1.18.1.linux-amd64.tar.gz
echo 'export GOROOT=/usr/local/go' >> $HOME/.bash_profile
echo 'export GOPATH=$HOME/go' >> $HOME/.bash_profile
echo 'export GO111MODULE=on' >> $HOME/.bash_profile
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile && . $HOME/.bash_profile
go version
```
### Install essentials:
```bash
cd $HOME
sudo apt update
sudo apt install make build-essential git jq ncdu bsdmainutils nload -y < "/dev/null"
```
### Install `archwayd`:
```bash
cd $HOME
rm -r $HOME/archway
git clone https://github.com/archway-network/archway
cd archway
git checkout main
make build
cp $HOME/archway/build/archwayd /usr/local/bin
archwayd version
```
### Configure your node:
```bash
archwayd init ${ARCHWAY_NODENAME} --chain-id torii-1
wget -O $HOME/.archway/config/genesis.json "https://raw.githubusercontent.com/archway-network/testnets/main/torii-1/genesis.json"
sha256sum $HOME/.archway/config/genesis.json 
# 0257e0be068328514f79ddcc1a3f8703909c116c6bd4a8dc1c9f8f31018e0d93
archwayd unsafe-reset-all
external_address=`curl ifconfig.me`
peers="facf38daac7cbbdcbaf87f531225d6a621cea483@15.235.10.78:26656,83b18e67dca836a838361496a7c87696a488fd05@65.108.99.224:26656,07fd2c5b8838dfc80ff1e9c5577006b552fcb98c@206.221.181.234:46656,c5ca4cb89df8c194e6b404f54be0e27c1258377b@95.214.55.210:26756,ece6b901c278f91410b798edef805ba1d358c660@59.13.223.197:30273,b1cedcd284964d7657d597541ec9516fa3392cd1@185.234.69.139:26656,ce1e6c7a84ab3f8e2fd87d4aef0f95da774a5e98@159.69.11.174:26656,cb1534d2ad2fedb1168b4052f04ede5b12428068@51.250.111.252:26656,2b0c484615d9bafd6cc339c588e366dd9b000221@54.180.95.251:26656,2e422fe3956b7ea2a868dbe832e8cd9af5203ea6@65.108.75.32:26656"
sed -i.bak -e "s/^external_address = \"\"/external_address = \"$external_address:26656\"/; s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.archway/config/config.toml
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0utorii\"/;" $HOME/.archway/config/app.toml
archwayd unsafe-reset-all
```
### Run `archwayd` through `systemd`:
```bash
echo "[Unit]
Description=Archway Node
After=network.target

[Service]
User=$USER
Type=simple
ExecStart=$(which archwayd) start
Restart=on-failure
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target" > $HOME/archwayd.service
sudo mv $HOME/archwayd.service /etc/systemd/system
sudo tee <<EOF >/dev/null /etc/systemd/journald.conf
Storage=persistent
EOF
sudo systemctl restart systemd-journald
sudo systemctl daemon-reload
sudo systemctl enable archwayd
sudo systemctl restart archwayd
journalctl -u archwayd -f
```
### Generate keys
If you dont have keys, you can generate them:
```bash
archwayd keys add wallet
```
### Create validator:
Change $ARCHWAY_VALIDATOR_NAME to your validator name.
```bash
archwayd tx staking create-validator \
--from wallet \
--amount 1000000utorii \
--min-self-delegation 1000000 \
--commission-rate 0.1 \
--commission-max-rate 0.2 \
--commission-max-change-rate 0.01 \
--pubkey $(archwayd tendermint show-validator) \
--chain-id=torii-1 \
--moniker $ARCHWAY_VALIDATOR_NAME
```



## Explorer
The explorer is available here: https://archway.explorers.guru/
