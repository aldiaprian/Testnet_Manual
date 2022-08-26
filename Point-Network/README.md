# Point Network Testnet


## Specification VPS

CPU : 4 or more physical CPU cores
RAM : 32 GB
Storage : 500GB SSD
Connection : 100 Mbps
OS : Ubuntu 18.04 +

## Explorer
http://e.trxlfsnxl.xyz/point

# Langsung ke TOUR nya

## Instal Otomatis
```
wget -qO start.sh https://raw.githubusercontent.com/aldiaprian/Testnet_Manual/master/Point-Network/start.sh && chmod +x start.sh && ./start.sh
```

## Setelah menginstal silakan Muat Variabel! (Post Installation)
```
source $HOME/.bash_profile
```
### Check info Sync

Catatan: Anda harus menyinkronkan ke blok terbaru, periksa status sinkronisasi dengan perintah ini
```
evmosd status 2>&1 | jq .SyncInfo
```

## Agar cepat False
Note: Highlite bakal ke 0 lalu tunggu 3 menit agar false, setelah itu cek lagi
```
systemctl stop evmosd
evmosd tendermint unsafe-reset-all --home $HOME/.evmosd
SEEDS=""
PEERS="e40b9738c23934abf2f34ba8091a48cd31f5a844@51.11.180.20:18656"; \
sed -i.bak -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.evmosd/config/config.toml
SNAP_RPC="http://51.11.180.20:18657"
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)
echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.evmosd/config/config.toml
systemctl restart evmosd && journalctl -u evmosd -f -o cat
```

## Buat dompet

Untuk membuat dompet baru Anda dapat menggunakan perintah di bawah ini Masukan Pharse Metamask Kalian dan Jangan lupa simpan mnemonicnya Validator

```
evmosd keys add $WALLET
```

(OPSIONAL) Untuk memulihkan dompet Anda menggunakan frase seed

```
evmosd keys add $WALLET --recover
```

Untuk mendapatkan daftar dompet saat ini

```
evmosd keys list
```
## Save Info Wallet

```
EVMOS_WALLET_ADDRESS=$(evmosd keys show $WALLET -a)
```
Masukan Pharse Wallet
```
EVMOS_VALOPER_ADDRESS=$(evmosd keys show $WALLET --bech val -a)
```
Masukan Pharse Wallet
```
echo 'export EVMOS_WALLET_ADDRESS='${EVMOS_WALLET_ADDRESS} >> $HOME/.bash_profile
echo 'export EVMOS_VALOPER_ADDRESS='${EVMOS_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```



## Minta Faucet Menggunakan Address Metamask (Kalo Udah Sekip)

- Isi Form : https://pointnetwork.io/testnet-form (Tunggu 24 Jam Akan Dapat Email dan Coin Test Masuk ke Metamask)
- Add RPC di Metamask (Untuk Memastikan Udah Ada Faucet Landing)

```
Network Title: Point XNet Triton
RPC URL: https://xnet-triton-1.point.space/
Chain ID: 10721
SYMBOL: XPOINT
```

## Cara Convert Token

Note : Jika Saldo faucet Yang kalian Punya Berbeda Address atau Ada di Wallet Metamask Yang Berbeda.
Intinya, Untuk Kalian Yang Claim Token Faucet Pakai Wallet Pertama Metamask dan Wallet Validator itu Berbeda..

## Export Private Key Validator kalian Dengan Perintah

```
evmosd keys unsafe-export-eth-key $WALLET
```

- Masukan Pharse atau Password Keyring (Sesuai Yang kalian Bikin Saat Buat Wallet)
- Salin Private Key nya
- Import ke Metamask
- Pindahkan dan Kirim Token XPOINT Yang ada di Address Pertama ke Address `0x..` Yang Baru kalian Import Tadi

## Convert XPOINT

- Buka Situs : https://evmos.me/utils/tools
- Connect Wallet
- Masukan Address `0x..` Metamaskan kalian di Addres Conventer anda Akan Melihat Address `Evmosxxxx` dan Pastikan Sama Dengan Address Wallet Validator Kalian
- Silahkan Check Ke Vps kalian dengan Perintah `evmosd query bank balances address-evmos-kalian`
- Maka Tara Saldo Anda Sudah Ada

## Buat Validator
### Check Saldo 

```
evmosd query bank balances $EVMOS_WALLET_ADDRESS
```
Jika Command di atas Error `$EVMOS_WALLET_ADDRESS` menjadi `Address Kalian`

## Create Validator Nya

```
evmosd tx staking create-validator \
--amount=100000000000000000000apoint \
--pubkey=$(evmosd tendermint show-validator) \
--moniker="MASUKAN-NAMA-VALIDATOR" \
--chain-id=point_10721-1 \
--commission-rate="0.10" \
--commission-max-rate="0.20" \
--commission-max-change-rate="0.01" \
--min-self-delegation="100000000000000000000" \
--gas="400000" \
--gas-prices="0.025apoint" \
--from=MASUKAN-ADDRESS-EVMOS \
--yes
```
**Penting :** Jika Output Yang keluar `code:32` atau `code:19` Artinya Error, kalian Bisa Restart Node Dengan Perintah `sudo systemctl restart evmosd && sudo journalctl -u evmosd -f -o cat` Ulangi ULang Buat Validato, Jika tx Sudah `code:0` Next Step
## Memantau validator Anda
Salin Txhash Nya Lalu Jalankan
```
evmosd query tx PASTE-TX-HASH-DI-SINI
```
Jika transaksi benar, Anda harus langsung menjadi bagian dari set validator. Periksa pubkey Anda terlebih dahulu:
```
evmosd tendermint show-validator
```
Anda akan melihat kunci di sana, Anda dapat mengidentifikasi simpul Anda di antara validator lain menggunakan kunci itu:
```
evmosd query tendermint-validator-set
```
Di sana Anda akan menemukan lebih banyak info seperti VotingPower Anda yang seharusnya lebih besar dari 0. Anda juga dapat memeriksa VotingPower Anda dengan menjalankan:
```
evmosd status
```

## Useful Commands
Check Logs
```
journalctl -fu evmosd -o cat
```
Start Service
```
sudo systemctl start evmosd
```
Stop Service
```
sudo systemctl stop evmosd
```
Restart Service
```
sudo systemctl restart evmosd
```
## Node Info
Synchronization info
```
evmosd status 2>&1 | jq .SyncInfo
```
Validator Info
```
evmosd status 2>&1 | jq .ValidatorInfo
```
Node Info
```
evmosd status 2>&1 | jq .NodeInfo
```
## Delegation, Dll
Untuk mendelegasikan ke validator Anda jalankan perintah ini: Catatan: Ubah ke suka Anda, misalnya: 1000000000000000000apoint adalah 100point
```
evmosd tx staking delegate $(evmosd tendermint show-address) <ammount>apoint --chain-id=point_10721-1 --from=<evmosvaloper> --gas=400000 --gas-prices=0.025apoint 
```
Ubah `<evmosvaloper>` ke alamat valoper Anda Untuk memeriksa alamat valoper jalankan perintah ini:
```
evmosd debug addr <evmos address>
```
## Manajemen Validator
Unjail Validator (PASTIKAN ANDA SYNCED DENGAN NODE TERBARU!!)
```
evmosd tx slashing unjail --from=$WALLET --chain-id=point_10721-1 --gas-prices=0.025apoint
```
Periksa apakah validator Anda aktif: (jika output tidak kosong, Anda adalah validator)
```
evmosd query tendermint-validator-set | grep "$(evmosd tendermint show-address)"
```
Lihat status slashing: (jika Jailed hingga tahun 1970 berarti Anda tidak dipenjara!)
```
evmosd query slashing sign-info $(evmosd tendermint show-validator)
```
## Claim Reward Hasil Validator

```
evmosd tx distribution withdraw-rewards VALOPER_ADDRESS-KALIAN --from=$WALLET --commission --chain-id=$EVMOS_CHAIN_ID
```
```
evmosd tx distribution withdraw-all-rewards --from=$WALLET --chain-id=$EVMOS_CHAIN_ID --gas=auto
```

## Hapus Node Secara Permanen (Cadangkan kunci Pribadi Anda terlebih dahulu jika Anda ingin bermigrasi !!)
```
sudo systemctl stop evmosd
sudo systemctl disable evmosd
sudo rm /etc/systemd/system/evmos* -rf
sudo rm $(which evmosd) -rf
sudo rm $HOME/.evmosd -rf
sudo rm $HOME/point-chain -rf
```
