# Relayer-icq

### hata aldıysanız önce bunu
```
systemctl stop icqd 
systemctl disable icqd
rm /etc/systemd/system/icqd.service* -rf
rm -rf /usr/local/bin/icq 
rm $(which icq) -rf 
rm -rf $HOME/.icq 
rm -rf $HOME/interchain-queries
```

### check go

```bash
# check go version is 1.18.x
cd $HOME
wget -O go1.18.2.linux-amd64.tar.gz https://go.dev/dl/go1.18.2.linux-amd64.tar.gz
rm -rf /usr/local/go && tar -C /usr/local -xzf go1.18.2.linux-amd64.tar.gz && rm go1.18.2.linux-amd64.tar.gz
echo 'export GOROOT=/usr/local/go' >> $HOME/.bashrc
echo 'export GOPATH=$HOME/go' >> $HOME/.bashrc
echo 'export GO111MODULE=on' >> $HOME/.bashrc
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bashrc && . $HOME/.bashrc
go version
# 1.18.*
```

### build

```bash
cd $HOME
git clone https://github.com/Stride-Labs/interchain-queries.git
cd interchain-queries
go build
sudo mv interchain-queries /usr/local/bin/icq
```

### config

```bash
cd $HOME

mkdir $HOME/.icq

tee $HOME/.icq/config.yaml > /dev/null <<EOF
default_chain: stride
chains:
  gaia:
    key: wallet
    chain-id: GAIA
    rpc-addr: **http://95.214.55.4:26667**   #ip adress
    grpc-addr: **http://95.214.55.4:9092**   #ip adress
    account-prefix: cosmos
    keyring-backend: test
    gas-adjustment: 1.2
    gas-prices: 0.001uatom
    key-directory: $HOME/.icq/keys
    debug: true
    timeout: 20s
    block-timeout: ""
    output-format: json
    sign-mode: direct
  stride:
    key: wallet
    chain-id: STRIDE-TESTNET-4
    rpc-addr: **http://127.0.0.1:26677**   #ip adress
    grpc-addr: **http://127.0.0.1:9094**   #ip adress
    account-prefix: stride
    keyring-backend: test
    gas-adjustment: 1.2
    gas-prices: 0.001ustrd
    key-directory: $HOME/.icq/keys
    debug: true
    timeout: 20s
    block-timeout: ""
    output-format: json
    sign-mode: direct
cl: {}
EOF

# restore wallets
icq keys restore --chain stride wallet <<< "stride mnemonic here"
icq keys restore --chain gaia wallet <<< "gaia mnemonic here"

```

### Create unit

```bash
tee $HOME/icqd.service > /dev/null <<EOF
[Unit]
Description=icq
After=network.target
[Service]
Type=simple
User=$USER
ExecStart=$(which icq) run --debug
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```
```
sudo mv $HOME/icqd.service /etc/systemd/system/
```
# start service
```
sudo systemctl daemon-reload
sudo systemctl enable icqd
```
# start icq
```
sudo systemctl restart icqd && journalctl -u icqd -f -o cat
```
**# WAIT 10-15 min before ANY logs will run**
