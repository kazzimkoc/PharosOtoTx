Pharos Testnet Otomatik Token Transfer Bot Kurulumu ve Scripti 
Şimdi sunucumuza geçip kurulumumuzu yapalım
```
cd 
nano .env
```
```
RPC_URL=https://testnet.dplabs-internal.com
PRIVATE_KEY=
RECEIVER_ADDRESS=0x
```

ctrl+x y Enter

Girişinizin doğrulunu kontrol edin
```
cat .env
```
```
nano pharostrans.py
```
Açılan dosyanıza aşağıdaki kodu yapıştırın. Yalnız Kodu inceleyin alıcı adresini ekleyin 

```
import os
import time
from web3 import Web3
from dotenv import load_dotenv

# Ortam değişkenlerini yükle
load_dotenv()

RPC_URL = os.getenv("RPC_URL")
PRIVATE_KEY = os.getenv("PRIVATE_KEY")
RECEIVER_ADDRESS = os.getenv("RECEIVER_ADDRESS")

if not RPC_URL or not PRIVATE_KEY or not RECEIVER_ADDRESS:
    print("RPC_URL, PRIVATE_KEY veya RECEIVER_ADDRESS .env dosyasında eksik!")
    exit()

# Web3 bağlantısı
web3 = Web3(Web3.HTTPProvider(RPC_URL))

if not web3.is_connected():
    print("Bağlantı başarısız! RPC URL kontrol edin.")
    exit()

SENDER_ADDRESS = web3.eth.account.from_key(PRIVATE_KEY).address
print(f"Bağlantı başarılı! Gönderici adresi: {SENDER_ADDRESS}")

# Burada native PHRS token transferi yapılıyor
# Gönderilecek miktar (0.0001 PHRS)
AMOUNT_TO_SEND = web3.to_wei(0.0001, 'ether')

def send_native_transfer():
    nonce = web3.eth.get_transaction_count(SENDER_ADDRESS)

    tx = {
        'nonce': nonce,
        'to': Web3.to_checksum_address(RECEIVER_ADDRESS),
        'value': AMOUNT_TO_SEND,
        'gas': 21000,
        'gasPrice': web3.to_wei(1, 'gwei'),
        'chainId': web3.eth.chain_id
    }

    signed_tx = web3.eth.account.sign_transaction(tx, PRIVATE_KEY)
    tx_hash = web3.eth.send_raw_transaction(signed_tx.raw_transaction)
    print(f"PHRS transfer işlemi gönderildi! Tx hash: {web3.to_hex(tx_hash)}")

INTERVAL = 60  # saniye

while True:
    try:
        send_native_transfer()
    except Exception as e:
        print(f"Gönderimde hata oluştu: {e}")
    time.sleep(INTERVAL)

```
ctrl + x y Enter
Virtual environment oluşturalım.
```
python3 -m venv venv
source venv/bin/activate
```
Gerekli kütüphaneleri yükleyelim.
```
pip install web3 python-dotenv
```
Screen açalım
```
screen -S pharostrans
```
```
python3 pharostrans.py
```
Biraz bekleyin txleriniz akmaya başlayacaktır. Her 60 sn yede bir sırası ile tx atacaktır.
Screen den çıkma için  ctrl + a + d  kullanınız. 
