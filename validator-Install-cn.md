# Celestia-Node-Mocha-3 验证节点创建教程
## **简介**
* 设置验证者节点您需满足以下硬件最低要求：
* 内存：8 GB RAM
* CPU：六核
* 磁盘：500 GB SSD 存储
* 带宽：下载 1 Gbps/上传 100 Mbps
* 验证者节点需使用内部 Celestia Core 节点运行 Celestia App 守护程序，且确保至少有 100+ Gb 的可用空间来安全地安装和运行验证者节点，验证者节点允许您参与 Celestia 网络中的共识。
## **一. 环境安装包下载**
```shell
sudo apt update && sudo apt upgrade -y
```

**回车继续下载**
```shell
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential git make ncdu -y
```

 **安装GO**
```shell
ver="1.20.2"
cd $HOME
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
```
**设置保存**
```shell
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.zshrc
source $HOME/.zshrc
```
**查询**
```shell
go version
```
## **二.  下载celestia-app**
```shell
cd $HOME
rm -rf celestia-app
git clone https://github.com/celestiaorg/celestia-app.git
cd celestia-app/
APP_VERSION=v1.0.0-rc9
git checkout tags/$APP_VERSION -b $APP_VERSION
make install
```
**克隆网络存储库**
```shell
cd $HOME
rm -rf networks
git clone https://github.com/celestiaorg/networks.git
```
**设置种子和对等点**
```shell
celestia-appd init "node-name" --chain-id mocha-3
cp $HOME/networks/mocha-3/genesis.json $HOME/.celestia-app/config
SEEDS="9aa8a73ea9364aa3cf7806d4dd25b6aed88d8152@celestia-testnet.seed.mzonder.com:11156,258f523c96efde50d5fe0a9faeea8a3e83be22ca@seed.mocha-3.celestia.aviaone.com:20275,3314051954fc072a0678ec0cbac690ad8676ab98@65.108.66.220:26656"
sed -i "s|^seeds *=.*|seeds = \"$SEEDS\"|" $HOME/.celestia-app/config/config.toml
PEERS="ec11f3be74010b78882de2cbd170d7ad4458d8ac@157.245.250.63:26656,ec11f3be74010b78882de2cbd170d7ad4458d8ac@157.245.250.63:26656,5073ad517afaeae51cf939381d7dce09880f47f6@51.158.76.4:26656,9c94e40188137f9b1314c205dd9a25fa62b7688e@198.27.82.6:26656,5ec7477a55b48984ec778bd1bef87d2ac8cf95eb@138.201.60.238:26656,e409695e0fcd1a2b521bae27621dca4686cb1ccc@198.199.70.16:26656"
sed -i -e "s|^persistent_peers *=.*|persistent_peers = \"$PEERS\"|" $HOME/.celestia-app/config/config.toml
sed -i 's/^timeout_commit *=.*/timeout_commit = "11s"/' $HOME/.celestia-app/config/config.toml
```

**配置空间修剪**
```shell
PRUNING="custom"
PRUNING_KEEP_RECENT="100"
PRUNING_INTERVAL="10"

sed -i -e "s/^pruning *=.*/pruning = \"$PRUNING\"/" $HOME/.celestia-app/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \
\"$PRUNING_KEEP_RECENT\"/" $HOME/.celestia-app/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \
\"$PRUNING_INTERVAL\"/" $HOME/.celestia-app/config/app.toml
```
**重置网络**
```shell
celestia-appd tendermint unsafe-reset-all --home $HOME/.celestia-app
```

**下载块数据**
```shell
cd $HOME
rm -rf ~/.celestia-app/data
mkdir -p ~/.celestia-app/data
SNAP_NAME=$(curl -s https://snaps.qubelabs.io/celestia/ | \
    egrep -o ">mocha-3.*tar" | tr -d ">")
wget -O - https://snaps.qubelabs.io/celestia/${SNAP_NAME} | tar xf - \
    -C ~/.celestia-app/data/
```

## **三. 创建Systemd**
```shell
sudo tee <<EOF >/dev/null /etc/systemd/system/celestia-appd.service
[Unit]
Description=celestia-appd Cosmos daemon
After=network-online.target

[Service]
User=$USER
ExecStart=$HOME/go/bin/celestia-appd start
Restart=on-failure
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
EOF
```
**启动查看实时节点状态**
```shell
sudo systemctl enable celestia-appd
sudo systemctl start celestia-appd
sudo journalctl -u celestia-appd.service -f
```
**日志查看**
```shell
sudo systemctl status celestia-appd
```
按ctlr+c停止
**检查您的节点是否同步到最新块（显示"catching_up": false就是最新块）**
```shell
curl -s localhost:26657/status | jq .result | jq .sync_info
```

## **四. 创建钱包（注意钱包地址名称等修改为自己的）**
```shell
celestia-appd keys add Olo
```

**或者导入旧钱包**
```shell
celestia-appd config keyring-backend test
celestia-appd keys add Olo --recover
```

**然后指定**
```shell
CELESTIA_ADDR=$(celestia-appd keys show celestia1d3ffkvga38eehxcnnmduskkpucahr0h8whpdx3 -a) 
echo $CELESTIA_ADDR 
echo 'export CELESTIA_ADDR='${CELESTIA_ADDR} >> $HOME/.bash_profile
```

**保存**
```shell
CELESTIA_VALOPER=$(celestia-appd keys show celestia1d3ffkvga38eehxcnnmduskkpucahr0h8whpdx3 --bech val -a) 
echo $CELESTIA_VALOPER 
echo 'export CELESTIA_VALOPER='${CELESTIA_VALOPER} >> $HOME/.bash_profile 
source $HOME/.bash_profile
```

**查询钱包余额**
 ```shell
celestia-appd q bank balances $CELESTIA_ADDR
```
**设置EVM地址（请修改为自己以太坊钱包地址）**
```shell
ERC20_ADDRESS=0x89B50fDfC146ee89C789A38482A5277778Da3C3c
```

**保存**
```shell
echo "export EVM_ADDRESS=""$ERC20_ADDRESS" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

**设置你的验证节点名称还有钱包名称**
```shell
CELESTIA_NODENAME="Olo666" CELESTIA_WALLET="Olo_WALLET"
CELESTIA_CHAIN="mocha-3"
```

**保存**
```shell
echo 'export CELESTIA_CHAIN='$CELESTIA_CHAIN >> $HOME/.bash_profile
echo 'export CELESTIA_NODENAME='${CELESTIA_NODENAME} >> $HOME/.bash_profile
echo 'export CELESTIA_WALLET='${CELESTIA_WALLET} >> $HOME/.bash_profile
source $HOME/.bash_profile
```
**配置客户端**
```shell
celestia-appd config chain-id $CELESTIA_CHAIN --home $HOME/.celestia-app
celestia-appd config keyring-backend test
```
**查询是否配置正确**
```shell
echo $CELESTIA_NODENAME,$CELESTIA_CHAIN,$CELESTIA_WALLET,$EVM_ADDRESS | tr "," "\n" | nl
```

**输入之后输出为**

* 1.您的验证者节点名称 Olo666

* 2.目前的测试网名称 mocha-3

* 3.您的钱包名称 Olo_wallet

* 4.您的以太坊地址 0xxxxx

**创建验证者**
```shell
celestia-appd tx staking create-validator \
--amount=1000000utia \
--pubkey=$(celestia-appd tendermint show-validator) \
--moniker=$CELESTIA_NODENAME \
--chain-id=$CELESTIA_CHAIN \
--commission-rate=0.1 \
--commission-max-rate=0.2 \
--commission-max-change-rate=0.01 \
--min-self-delegation=1000000 \
--from=Olo \
--evm-address=$EVM_ADDRESS \
  --gas auto \
  --gas-adjustment 1.3 \
  --fees 45000utia
```
  最后，然后回复Y即可创建(如果fees不够请增加fees)
  

## 一些有用的命令


* 修改验证器名字
```shell
celestia-appd tx staking edit-validator --new-moniker="Olo233" --chain-id=mocha-3 --from="Olo" --fees 21000utia
```
* 编辑验证者头像，注册https://keybase.io/在里面编辑头像 然后输入以下命令
 ```shell
 celestia-appd tx staking edit-validator --identity "07C2 AA9B 6741 E176" --from=Olo  --fees 21000utia
  ```
* 编辑验证器网站还有简介
```shell
celestia-appd tx staking edit-validator --details="Community builders" --website="https://mobile.twitter.com/Ting_0x0" --from=Olo  --fees 21000utia
```
* 质押
```shell
celestia-appd tx staking delegate celestiavaloper1d3ffkvga38eehxcnnmduskkpucahr0h8tgr5sh 95000000utia --from=Olo --fees 300utia --chain-id=mocha-3 --node https://rpc.mocha-3.celestia.aviaone.com:443
 ```
