# rebus
**Clone project repository**
```
cd && rm -rf rebus.core
git clone https://github.com/rebuschain/rebus.core
cd rebus.core
git checkout v0.5.0
```

**Build binary**
```
make install
```

**Set node CLI configuration**
```
rebusd config chain-id reb_1111-1
rebusd config keyring-backend file
rebusd config node tcp://localhost:17257
```
# Initialize the node
rebusd init "Your Node Name" --chain-id reb_1111-1

# Download genesis and addrbook files
curl -L https://snapshots.nodejumper.io/rebus/genesis.json > $HOME/.rebusd/config/genesis.json
curl -L https://snapshots.nodejumper.io/rebus/addrbook.json > $HOME/.rebusd/config/addrbook.json

# Set seeds
sed -i -e 's|^seeds *=.*|seeds = "718706d1a1e93c2fe9a3059588236cf96c457ff4@seed.rebus.cros-nest.com:26656,20e1000e88125698264454a884812746c2eb4807@seeds.lavenderfive.com:17256,ebc272824924ea1a27ea3183dd0b9ba713494f83@rebus-mainnet-seed.autostake.com:26906,8542cd7e6bf9d260fef543bc49e59be5a3fa9074@seed.publicnode.com:26656,0863966356f6532377aeba663415258d44ddbd13@rebus.peer.stavr.tech:40106,ebc272824924ea1a27ea3183dd0b9ba713494f83@rebus-mainnet-peer.autostake.com:26906,9c7c067bd73bddfe8da39087cdae37c4fc5ec6e3@5.9.69.107:26656"|' $HOME/.rebusd/config/config.toml

# Set minimum gas price
sed -i -e 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.01arebus"|' $HOME/.rebusd/config/app.toml

# Set pruning
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "17"|' \
  $HOME/.rebusd/config/app.toml

# Change ports
sed -i -e "s%:1317%:17217%; s%:8080%:17280%; s%:9090%:17290%; s%:9091%:17291%; s%:8545%:17245%; s%:8546%:17246%; s%:6065%:17265%" $HOME/.rebusd/config/app.toml
sed -i -e "s%:26658%:17258%; s%:26657%:17257%; s%:6060%:17260%; s%:26656%:17256%; s%:26660%:17261%" $HOME/.rebusd/config/config.toml

# Download latest chain data snapshot
curl "https://snapshots.nodejumper.io/rebus/rebus_latest.tar.lz4" | lz4 -dc - | tar -xf - -C "$HOME/.rebusd"

# Create a service
sudo tee /etc/systemd/system/rebusd.service > /dev/null << EOF
[Unit]
Description=Rebus node service
After=network-online.target
[Service]
User=$USER
ExecStart=$(which rebusd) start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable rebusd.service

# Start the service and check the logs
sudo systemctl start rebusd.service
sudo journalctl -u rebusd.service -f --no-hostname -o cat
