# **一些常用命令**
## 关于systemctl命令
* 检查实时状态
    ```shell
    journalctl -u celestia-xxx.service -f
    ```
* 停止节点
    ```shell
   systemctl stop celestia-xxx
    ```
* 启动节点
```shell
systemctl start celestia-xxx
  ```
* 重启节点
 ```shell
systemctl restart celestia-xxx
 ```
* 查看日志
```shell
systemctl status celestia-xxx
 ```
* 在开机时启用一个服务
```shell
systemctl enable celestia-xxx.service
```
* 开机的时候禁用该服务
```shell
systemctl disable elestia-xxx.service
```
* 查看已经启用的服务
```shell
systemctl list-unit-files|grep enabled
```
* 查看没有被激活的程序
```shell
systemctl list-unit-files
```
* 进入systemd编辑
```shell
# systemd管理的unit也就是后缀是service的文件存放的路径为：/etc/systemd/system/或者/usr/lib/systemd/system/
sudo nano /etc/systemd/system/celestia-xxx.service
```
* 检查安装路径
```shell
cat /etc/systemd/system/celestia-xxx.service
```
* 服务进程创建
```shell
sudo tee <<EOF >/dev/null /etc/systemd/system/celestia-xxx.service
[Unit]
Description=celestia-xxx Light Node
After=network-online.target

[Service]
User=$USER
ExecStart=/usr/local/bin/celestia light start --core.ip https://rpc.mocha.celestia.counterpoint.software --core.grpc.port 9090 --gateway --gateway.addr 127.0.0.1 --keyring.accname Abc --gateway.port 26659 --p2p.network mocha
Restart=on-failure
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
EOF
```
## 关于端口
* 端口开启
```shell
sudo ufw allow 26657/tcp
```
* 启动端口
```shell
sudo ufw enable
```
* 检查端口状态
```shell
sudo ufw status
```
* 重要提示：如果您要远程访问此服务器，请确保在启用之前启用 SSHufw
```shell
sudo ufw allow ssh
```
## 关于升级删除
* 升级go
```shell
ver="1.20.3" 
cd $HOME 
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" 
sudo rm -rf /usr/local/go 
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" 
rm "go$ver.linux-amd64.tar.gz"
```
* 保存
```shell
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
```
* 升级节点
```shell
cd celestia-node
git fetch --tags 
git checkout v0.10.4
make build 
sudo make install
```
* 删除GO
```shell
sudo rm -rf /usr/local/go
sudo rm -rf ~/go
 ```
* 删除节点
```shell
sudo systemctl stop celestia-full
sudo systemctl disable celestia-full
rm /etc/systemd/system/celestia-full.service
sudo systemctl daemon-reload
cd $HOME
rm -rf .celestia-full celestia-node
 ```
* 删除块存储数据
```shell
cd $HOME
cd .celestia-light-blockspacerace-0
sudo rm -rf blocks index data transients
```
# 关于节点
* 查询app版本
```shell
celestia-appd version
```
* 查询块存储
```shell
du -h ~/.celestia-bridge-blockspacerace-0
```
* id查询
```shell
NODE_TYPE=bridge
AUTH_TOKEN=$(celestia $NODE_TYPE auth admin --p2p.network blockspacerace )
```
* 继续输入以下命令查询
```shell
curl -X POST \
     -H "Authorization: Bearer $AUTH_TOKEN" \
     -H 'Content-Type: application/json' \
     -d '{"jsonrpc":"2.0","id":0,"method":"p2p.Info","params":[]}' \
     http://localhost:26658
```
* node钱包恢复
```shell
~/celestia-node/cel-key add Ting --keyring-backend test --node.type bridge --recover --p2p.network blockspacerace
```
* 自定义创建钱包（桥）
```shell
./cel-key add Abc --keyring-backend test --node.type bridge --p2p.network blockspacerace
```
* 轻节点钱包列表查询
```shell
~/celestia-node/cel-key list --node.type light --keyring-backend test --p2p.network blockspacerace
```
* 删除轻钱包
```shell
~/celestia-node/cel-key delete xxx --node.type light --keyring-backend test --p2p.network blockspacerace
```
* 转账
```shell
celestia-appd tx bank send celestia1f8ph7glalzulppv7jkp82087djrah5lszzu9hh celestia1r08gm89kqm4vxx70cjfdnmy4zdrhvykvfpncs0 3000000utia --fees 300utia --node https://rpc-blockspacerace.pops.one:443 --chain-id blockspacerace-0
* 质押
```shell
celestia-appd tx staking delegate celestiavaloper1d3ffkvga38eehxcnnmduskkpucahr0h8tgr5sh 95000000utia --from=Olo --fees 300utia --chain-id=mocha-3 --node https://rpc.mocha-3.celestia.aviaone.com:443
 ```
* 查询app钱包
```shell
celestia-appd keys list
```
* 重新改名钱包名称
```shell
celestia-appd keys rename Ting <NEW_KEY_NAME>
```
* 删除app钱包
```shell
celestia-appd keys delete Ting
```
* 进入节点目录
```shell
cd celestia-node/
```
## 关于验证者
* 提取所有验证者奖励
```shell
celestia-appd tx distribution withdraw-all-rewards --from Ting --chain-id mocha --gas-adjustment 1.4 --gas auto --fees 1000utia -y
```
* 提取自己佣金奖励
```shell
celestia-appd tx distribution withdraw-rewards $(celestia-appd keys show wallet --bech val -a) --commission --from wallet --chain-id mocha --gas-adjustment 1.4 --gas auto --fees 1000utia -y
```
* 解除自己的验证器质押
```shell
celestia-appd tx staking unbond $(celestia-appd keys show wallet --bech val -a) 1000000utia --from wallet --chain-id mocha --gas-adjustment 1.4 --gas auto --fees 1000utia -y
```
* 解除质押
```shell
celestia-appd tx staking unbond celestiavaloper1d3ffkvga38eehxcnnmduskkpucahr0h8tgr5sh 23000000utia --from=Ting --fees 300utia --chain-id=mocha --node https://rpc-mocha.pops.one:443
```
* persistent_peers对等点添加
```shell
persistent_peers="2d66c6323c20c730d9b75773a94eaf802195c771@34.143.250.237:26656,dc9e20553d0ac00d0bbd373caeb91256d9e99236@80.240.31.236:266566"
```
* 保存对等点添加
```shell
sed -i "s/^persistent_peers *=.*/persistent_peers = \"$persistent_peers\"/" $HOME/.celestia-app/config/config.toml
```
* 种子添加
```shell
SEEDS="ade4d8bc8cbe014af6ebdf3cb7b1e9ad36f412c0@testnet-seeds.polkachu.com:11656"
```
* 保存种子添加
```shell
sed -i "s|^seeds *=.*|seeds = \"$SEEDS\"|" $HOME/.celestia-app/config/config.toml
```* 修改验证器名字
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
* 查询验证者地址
```shell
celestia-appd keys show celestia1d3ffkvga38eehxcnnmduskkpucahr0h8whpdx3 --bech val -a
```
* 创建+编辑资料
```shell
celestia-appd tx staking create-validator \
--amount=900000utia \
--pubkey=$(celestia-appd tendermint show-validator) \
--moniker=$CELESTIA_NODENAME \
--identity="07C2 AA9B 6741 E176" \
--details="Community builders" \
--website="https://mobile.twitter.com/0xPandaJun" \
--chain-id=$CELESTIA_CHAIN \
--commission-rate=0.1 \
--commission-max-rate=0.2 \
--commission-max-change-rate=0.01 \
--min-self-delegation=1000000 \
--from=Ting \
--evm-address=$EVM_ADDRESS \
--orchestrator-address=$ORCHESTRATOR_ADDRESS \
--gas auto \
--gas-adjustment 1.3 \
--fees 1700utia
```
