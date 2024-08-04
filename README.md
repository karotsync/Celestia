**install go, if needed**
```
cd $HOME
VER="1.21.3"
wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm "go$VER.linux-amd64.tar.gz"
[ ! -f ~/.bash_profile ] && touch ~/.bash_profile
echo "export PATH=$PATH:/usr/local/go/bin:~/go/bin" >> ~/.bash_profile
source $HOME/.bash_profile
[ ! -d ~/go/bin ] && mkdir -p ~/go/bin
```
**set vars**
```
echo "export WALLET="wallet"" >> $HOME/.bash_profile
echo "export MONIKER="test"" >> $HOME/.bash_profile
echo "export CELESTIA_CHAIN_ID="mocha-4"" >> $HOME/.bash_profile
echo "export CELESTIA_PORT="11"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

**download binary**
```
cd $HOME 
rm -rf celestia-app 
git clone https://github.com/celestiaorg/celestia-app.git 
cd celestia-app/ 
APP_VERSION=v1.11.0
git checkout tags/$APP_VERSION -b $APP_VERSION 
make install
cd $HOME
rm -rf networks
git clone https://github.com/celestiaorg/networks.git
celestia-appd init "test" --chain-id $CELESTIA_CHAIN_ID
cp $HOME/networks/mocha-4/genesis.json $HOME/.celestia-app/config
sed -i -e "s|^target_height_duration *=.*|timeout_commit = \"11s\"|" $HOME/.celestia-app/config/config.toml
```

**config and init app**
```
celestia-appd config node tcp://localhost:${CELESTIA_PORT}657
celestia-appd config keyring-backend os
celestia-appd config chain-id mocha-4
celestia-appd init "test" --chain-id mocha-4
```

**download genesis and addrbook**
```
wget -O $HOME/.celestia-app/config/genesis.json https://server-4.itrocket.net/testnet/celestia/genesis.json
wget -O $HOME/.celestia-app/config/addrbook.json  https://server-4.itrocket.net/testnet/celestia/addrbook.json
```

**set seeds and peers**
```
SEEDS="5d0bf034d6e6a8b5ee31a2f42f753f1107b3a00e@celestia-testnet-seed.itrocket.net:11656"
PEERS="daf2cecee2bd7f1b3bf94839f993f807c6b15fbf@celestia-testnet-peer.itrocket.net:11656,bdbb36bb9afd57400635623268e93a6ea629a3dd@141.94.138.48:26679,13de82ffe31c99298350d3aabf253a8d47a3e0b2@125.253.92.7:26656,5a7566aa030f7e5e7114dc9764f944b2b1324bcd@65.109.23.114:11656,e726816f42831689eab9378d5d577f1d06d25716@164.152.163.148:36656,6bfb98f919b8e655cc99938a18140aa4cedf6518@88.218.224.72:26656,a8bb0bc58133022effcf415663d93c0ca835ebde@13.212.141.100:26656,a98484ac9cb8235bd6a65cdf7648107e3d14dab4@116.202.231.58:12056,43e9da043318a4ea0141259c17fcb06ecff816af@141.94.73.39:43656,c8a8e1882a4a8a977fc7e030b93cc7aaec3180b6@211.219.19.78:26656,36c18ca2aae40a8143ce84809092a64f99edb502@162.250.127.226:36656,6ed983017167d96c62b166725250940deb783563@65.108.142.147:27656,1feea4b8b29d6bb9c51dc29d0f2029db06b7219c@169.155.169.214:36656,70e8a8941f32dc5f696e46ee836c27620e773065@78.46.65.144:26656,c3d4e9daef6db3c1646ad7c4198b064361485299@65.109.126.24:26630,0770a8666d2d602d1c8e9ef055052f86bab06eac@147.135.144.57:26656,6cabdecd60b320c9481df4e63678623026283fab@136.243.94.113:26656,269d63adbe022d956fdd6735b93fc8e5b2fad5a8@93.190.143.6:26656,00144a5d9397384941a595346e05045cc7a9aef9@18.139.78.193:26656,16c5b4463706f49d2db19d3288516efc50582000@65.21.233.188:11656,de181ebe22ce14483abbb8695bdb43c1169246af@185.144.99.223:26656"
sed -i -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*seeds *=.*/seeds = \"$SEEDS\"/}" \
       -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*persistent_peers *=.*/persistent_peers = \"$PEERS\"/}" $HOME/.celestia-app/config/config.toml
```

**set custom ports in app.toml**
```
sed -i.bak -e "s%:1317%:${CELESTIA_PORT}317%g;
s%:8080%:${CELESTIA_PORT}080%g;
s%:9090%:${CELESTIA_PORT}090%g;
s%:9091%:${CELESTIA_PORT}091%g;
s%:8545%:${CELESTIA_PORT}545%g;
s%:8546%:${CELESTIA_PORT}546%g;
s%:6065%:${CELESTIA_PORT}065%g" $HOME/.celestia-app/config/app.toml
```

**set custom ports in config.toml file**
```
sed -i.bak -e "s%:26658%:${CELESTIA_PORT}658%g;
s%:26657%:${CELESTIA_PORT}657%g;
s%:6060%:${CELESTIA_PORT}060%g;
s%:26656%:${CELESTIA_PORT}656%g;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${CELESTIA_PORT}656\"%;
s%:26660%:${CELESTIA_PORT}660%g" $HOME/.celestia-app/config/config.toml
```

**config pruning**
```
sed -i -e "s/^pruning *=.*/pruning = \"custom\"/" $HOME/.celestia-app/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" $HOME/.celestia-app/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"50\"/" $HOME/.celestia-app/config/app.toml
```

**set minimum gas price, enable prometheus and disable indexing**
```
sed -i 's|minimum-gas-prices =.*|minimum-gas-prices = "0.002utia"|g' $HOME/.celestia-app/config/app.toml
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.celestia-app/config/config.toml
sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.celestia-app/config/config.toml
```

**create service file**
```
sudo tee /etc/systemd/system/celestia-appd.service > /dev/null <<EOF
[Unit]
Description=Celestia node
After=network-online.target
[Service]
User=$USER
WorkingDirectory=$HOME/.celestia-app
ExecStart=$(which celestia-appd) start --home $HOME/.celestia-app
Restart=on-failure
RestartSec=5
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```
**reset and download snapshot**
```
celestia-appd tendermint unsafe-reset-all --home $HOME/.celestia-app
if curl -s --head curl https://server-4.itrocket.net/testnet/celestia/celestia_2024-07-30_2373319_snap.tar.lz4 | head -n 1 | grep "200" > /dev/null; then
  curl https://server-4.itrocket.net/testnet/celestia/celestia_2024-07-30_2373319_snap.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.celestia-app
    else
  echo "no snapshot founded"
fi
```

**enable and start service**
```
sudo systemctl daemon-reload
sudo systemctl enable celestia-appd
sudo systemctl restart celestia-appd && sudo journalctl -u celestia-appd -f
```
