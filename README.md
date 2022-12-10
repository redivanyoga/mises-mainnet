# mises-mainnet #
node on mises mainnet
# Tutorial Runnig Validator Mises Mainnet #

<p align="center">
  <img height="auto" width="auto" src="https://user-images.githubusercontent.com/38981255/204093973-e07f75af-b7e1-4871-83b7-4fd2ad4cdff6.png">
</p>

## Referensi

[Dokumen resmi](https://github.com/mises-id/mises-tm)

[Server Discord ](https://discord.gg/Eh2j3Ddz)

[Form Register Validator](https://docs.google.com/forms/u/0/d/e/1FAIpQLSdTqmA_ZfzIB5rWPI_GVA5X7joPG7ZKI5pU-VreI3_BhY0dtQ/viewform?usp=form_confirm)

[Details Event Mainnet](https://www.mises.site/validator)

## Persyaratan hardware & software

### Persyaratan software/OS

| Komponen | Spesifikasi minimal |
|----------|---------------------|
|Sistem Operasi|Ubuntu 16.04|

| Komponen | Spesifikasi rekomendasi |
|----------|---------------------|
|Sistem Operasi|Ubuntu 20.04-22|

### Sebelum Menjalankan Kan Node, Pastikan Anda Memiliki Mises Mainnet (Karena Butuh Token Mainnet Bukan Testnet)

### Install otomatis

```
wget -O mises.sh https://raw.githubusercontent.com/Whalealert/mises-mainnet/main/mises.sh && chmod +x mises.sh && ./mises.sh```
```

### Load variable ke system

```
source $HOME/.bash_profile
```

### Statesync
``` 
sudo systemctl stop misestmd
SNAP_RPC="https://e1.mises.site:443"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" $HOME/.misestm/config/config.toml

sudo systemctl restart misestmd && sudo journalctl -u misestmd -f --no-hostname -o cat | grep chunk
```

### Informasi node


   * cek sync node
   
```
misestmd status 2>&1 | jq .SyncInfo
```

   * cek log node
   
```
journalctl -fu misestmd -o cat
```

   * cek node info

```
misestmd status 2>&1 | jq .NodeInfo
```

* cek validator info

```
misestmd status 2>&1 | jq .ValidatorInfo
```

* cek node id
```
misestmd tendermint show-node-id
```

### Membuat wallet
* wallet baru

```
misestmd keys add $WALLET
```
* recover wallet

```
misestmd keys add $WALLET --recover
```
* list wallet

```
misestmd keys list
```
   * hapus wallet
```
misestmd keys delete $WALLET
```
### Simpan informasi wallet
```
MISES_WALLET_ADDRESS=$(misestmd keys show $WALLET -a)
MISES_VALOPER_ADDRESS=$(misestmd keys show $WALLET --bech val -a)
echo 'export MISES_WALLET_ADDRESS='${MISES_WALLET_ADDRESS} >> $HOME/.bash_profile
echo 'export MISES_VALOPER_ADDRESS='${MISES_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### Membuat validator
 * cek balance
```
misestmd query bank balances $MISES_WALLET_ADDRESS
```
 * membuat validator
```
misestmd tx staking create-validator \
  --amount 1000000umis \
  --from $WALLET \
  --commission-max-change-rate "0.01" \
  --commission-max-rate "0.2" \
  --commission-rate "0.07" \
  --min-self-delegation "1" \
  --pubkey  $(misestmd tendermint show-validator) \
  --moniker $NODENAME \
  --fees 250umis \
  --chain-id $MISES_CHAIN_ID
```
 * edit validator
```
misestmd tx staking edit-validator \
  --moniker="nama-node" \
  --identity="<your_keybase_id>" \
  --website="<your_website>" \
  --details="<your_validator_description>" \
  --chain-id=$MISES_CHAIN_ID \
  --fees 250umis \
  --from=$WALLET
```
 ° unjail validator
```
misestmd tx slashing unjail \
  --broadcast-mode=block \
  --from=$WALLET \
  --chain-id=$MISES_CHAIN_ID \
  --fees=250umis
```
### Voting
```
misestmd tx gov vote 1 yes --from $WALLET --chain-id=$MISES_CHAIN_ID
```
### Delegasi dan Rewards
  * delegasi
```
misestmd tx staking delegate $MISES_VALOPER_ADDRESS 1000000umis --from=$WALLET --chain-id=MISES_CHAIN_ID --fees=250umis
```
  * withdraw reward
```
misestmd tx distribution withdraw-all-rewards --from=$WALLET --chain-id=$MISES_CHAIN_ID --fees=250umis
```
  * withdraw reward beserta komisi
```
misestmd tx distribution withdraw-rewards $MISES_VALOPER_ADDRESS --from=$WALLET --commission --chain-id=$MISES_CHAIN_ID
```

### Stop Node

```
sudo systemctl stop misestmd
```

### Restart Node

```
sudo systemctl restart misestmd
```

### Hapus node
```
sudo systemctl stop misestmd && \
sudo systemctl disable misestmd && \
rm /etc/systemd/system/misestmd.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf mises-tm && \
rm -rf mises.sh && \
rm -rf .misestm && \
rm -rf $(which misestmd)
```
