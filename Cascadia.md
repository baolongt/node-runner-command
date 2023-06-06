### [source](https://cascadia.gitbook.io/gitbook/validators/install-your-node)

_Đã sửa theo 1 số lỗi gặp phải_

1. update package

```shell
sudo apt update && sudo apt upgrade -y
sudo apt install make build-essential gcc git
```

2. cài đặt Go

```shell
wget https://golang.org/dl/go1.19.2.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.19.2.linux-amd64.tar.gz
```

3. export env

```shell
GOROOT=/usr/local/go
GOPATH=$HOME/go
GO111MODULE=on
PATH=$PATH:/usr/local/go/bin:$HOME/go/bin
```

4. update profile

```shell
source ~/.profile
```

5. Build Cascadia from source.

```shell
curl -L https://github.com/CascadiaFoundation/cascadia/releases/download/v0.1.1/cascadiad-v0.1.1-linux-amd64 -o cascadiad
sudo chmod u+x cascadiad
sudo cp cascadiad /usr/local/bin/cascadiad
```

bước này thì thay `<username>` bằng output của command `whoami` này

```shell
#get username
whoami
```

```shell
sudo chown <username> /usr/local/bin/cascadiad
```

ví dụ

```shell
sudo chown long /usr/local/bin/cascadiad
```

6. check version

```shell
cascadiad version
```

7. init chain

thay `[moniker]` bằng cái gì đó rồi note lại

```shell
cascadiad init [moniker] --chain-id cascadia_6102-1
```

8. download genesis file

```shell
curl -LO https://github.com/CascadiaFoundation/chain-configuration/raw/master/testnet/genesis.json.gz
gunzip genesis.json.gz
cp genesis.json ~/.cascadiad/config/
```

9. Set persistent peers

```shell
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$(curl  https://raw.githubusercontent.com/CascadiaFoundation/chain-configuration/master/testnet/persistent_peers.txt)\"/" ~/.cascadiad/config/config.toml
```

10. Set minimum gas price

```shell
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0025aCC\"/" ~/.cascadiad/config/app.toml
```

11. Create systemd service file

```shell
sudo nano /etc/systemd/system/cascadiad.service
```

12. sửa `<username>` thành tên lúc ở bước 5

```shell
[Unit]
Description=Cascadia Node
After=network.target

[Service]
Type=simple
User=<username>
WorkingDirectory=/usr/local/bin
ExecStart=/usr/local/bin/cascadiad start --trace --log_level info --json-rpc.api eth,txpool,personal,net,debug,web3 --api.enable
Restart=on-failure
StartLimitInterval=0
RestartSec=3
LimitNOFILE=65535
LimitMEMLOCK=209715200

[Install]
WantedBy=multi-user.target
```

> Bấm Ctrl + S để save và Ctrl + X để exit.

13. Start your Node

```shell
# reload service files
sudo systemctl daemon-reload

# create the symlink
sudo systemctl enable cascadiad.service

# start the node
sudo systemctl start cascadiad.service
```

cái này show logs thôi nhảy text hơi nhiều out ra bằng `ctrl + c`

```shell
# show logs
journalctl -u cascadiad -f
```

### Chạy validator

**Source**: [Run your validator](https://cascadia.gitbook.io/gitbook/validators/run-your-validator)

---

#####Lỗi hay gặp

> Error: post failed: Post "https://rpc.cascadia.foundation": dial tcp: address rpc.cascadia.foundation: missing port in address

lỗi này thì fix bằng cách thêm flag

```shell
--node="https://rpc.cascadia.foundation:443"
```

hoặc

```shell
-n https://rpc.cascadia.foundation:443
```

tuỳ theo command sử dụng

---

1. check status

```shell
cascadiad status -n https://rpc.cascadia.foundation:443
```

nó sẽ return về JSON tìm cái đoạn nào có chữ `catching_up` nếu là `false` thì synced rồi còn `true` thì chờ rồi chạy lại command

2. Generate and store your validator's address and keys.

tạo key, thay thế `<key_name>` bằng tên gì đó rồi note lại `wallet address` + `seed phrase`

```shell
cascadiad keys add <key_name>
```

bước này thì convert ngược địa chỉ ví `cascadia` thành `ox`

```shell
cascadiad address-converter <your_wallet_address>
```

3. faucet

ở [link](https://www.cascadia.foundation/faucet)
hoặc discord

4. kiểm tra coi ví đã có token chưa

[explorer](https://explorer.cascadia.foundation/)

5. Stake token

Cách phần nên sửa

`<key_name>`: Tên nhập ở bước 2.
`<validator_name>`: Tên của validator.
`<description>`: Diễn tả của validator.
`<email_address>`: Địa chỉ email.
`<your_website>`: Địa chỉ website.
`<token_delegation>`: Số lượng token muốn delegate (1 CC = 1,000,000,000,000,000,000 aCC)

```shell
cascadiad tx staking create-validator \
--from <key_name> \
--chain-id cascadia_6102-1 \
--moniker="<validator_name>" \
--commission-max-change-rate=0.01 \
--commission-max-rate=1.0 \
--commission-rate=0.05 \
--details="<description>" \
--security-contact="<email_address>" \
--website="<your_website>" \
--pubkey $(cascadiad tendermint show-validator) \
--min-self-delegation="1" \
--amount <token_delegation>aCC \
--gas "auto" \
--gas-adjustment=1.2 \
--gas-prices="7aCC" \
--broadcast-mode block \
--node="https://rpc.cascadia.foundation:443"
```

**Ví dụ** :

```
cascadiad tx staking create-validator \
--from yourvalidator \
--chain-id cascadia_6102-1 \
--moniker="ubuntu" \
--commission-max-change-rate=0.01 \
--commission-max-rate=1.0 \
--commission-rate=0.05 \
--details="The World's First  Neocybernetic  Blockchain" \
--security-contact="admin@cascadia.foundation" \
--website="cascadia.foundation" \
--pubkey $(cascadiad tendermint show-validator) \
--min-self-delegation="1" \
--amount 1000000000000000000aCC \
--gas "auto" \
--gas-adjustment=1.2 \
--gas-prices="7aCC" \
--broadcast-mode block \
--node="https://rpc.cascadia.foundation:443"
```


--- Node Status  `JAILED`

```shell
cascadiad tx slashing unjail --from <wallet address> --gas-prices="7aCC" --gas="116000" --gas-adjustment=1.2 --chain-id=cascadia_6102-1 --yes --node https://rpc.cascadia.foundation:443
```

---
