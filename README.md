# namada
Install Dependencies
```bash
sudo apt update && sudo apt upgrade -y
sudo apt-get install -y make git-core libssl-dev pkg-config libclang-12-dev build-essential protobuf-compiler
```

Install GO
```bash
cd $HOME
! [ -x "$(command -v go)" ] && {
VER="1.20.3"
wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm "go$VER.linux-amd64.tar.gz"
[ ! -f ~/.bash_profile ] && touch ~/.bash_profile
echo "export PATH=$PATH:/usr/local/go/bin:~/go/bin" >> ~/.bash_profile
source $HOME/.bash_profile
}
[ ! -d ~/go/bin ] && mkdir -p ~/go/bin
```

Install Rust
```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y
source $HOME/.cargo/env
```

Install Protocol Buffer
```bash
cd $HOME
curl -L -o protobuf.zip https://github.com/protocolbuffers/protobuf/releases/download/v24.4/protoc-24.4-linux-x86_64.zip
mkdir protobuf_temp && unzip protobuf.zip -d protobuf_temp/
sudo cp protobuf_temp/bin/protoc /usr/local/bin/
sudo cp -r protobuf_temp/include/* /usr/local/include/
rm -rf protobuf_temp protobuf.zip
```

Install CometBFT
```bash
cd $HOME
rm -rf cometbft
git clone https://github.com/cometbft/cometbft.git
cd cometbft
git checkout v0.37.2
make build
sudo cp $HOME/cometbft/build/cometbft /usr/local/bin/
cometbft version
```

Build Namada
```bash
cd $HOME
rm -rf public-testnet-15.0dacadb8d663
git clone -b v0.28.2 https://github.com/anoma/namada.git public-testnet-15.0dacadb8d663
cd public-testnet-15.0dacadb8d663
make build-release
for BIN in namada namadac namadan namadar namadaw; do install -m 0755 target/release/$BIN $HOME/.local/bin/$BIN; done
```

Create SystemD Service Unit
```bash
sudo tee /etc/systemd/system/namada.service > /dev/null << EOF
[Unit]
Description=Namada node
After=network-online.target

[Service]
User=$USER
ExecStart=$HOME/.local/bin/namada node ledger run
Restart=always
RestartSec=10
LimitNOFILE=65535
Environment="CMT_LOG_LEVEL=p2p:none,pex:error"
Environment="NAMADA_CMT_STDOUT=true"
Environment="NAMADA_LOG=info"
Environment="PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:$HOME/.local/bin"

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable namada.service
```

Initialize Namada Node
```bash
export PATH=$HOME/.local/bin:$PATH
namada client utils join-network --chain-id public-testnet-15.0dacadb8d663

export CUSTOM_PORT=266
sed -i \
  -e "s|^proxy_app = \"tcp://127.0.0.1:26658\"|proxy_app = \"tcp://127.0.0.1:${CUSTOM_PORT}58\"|" \
  -e "s|^laddr = \"tcp://127.0.0.1:26657\"|laddr = \"tcp://127.0.0.1:${CUSTOM_PORT}57\"|" \
  -e "s|^laddr = \"tcp://0.0.0.0:26656\"|laddr = \"tcp://0.0.0.0:${CUSTOM_PORT}56\"|" \
  -e "s|^prometheus_listen_addr = \":26660\"|prometheus_listen_addr = \":${CUSTOM_PORT}66\"|" \
  $HOME/.local/share/namada/public-testnet-15.0dacadb8d663/config.toml
```

Start Namada Services and Logs
```bash
sudo systemctl start namada.service && sudo journalctl -u namada.service -f --no-hostname -o cat
```

Check Namada Version
```bash
namada --version
```
