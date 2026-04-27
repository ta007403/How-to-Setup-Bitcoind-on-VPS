# How-to-Setup-Bitcoind-on-VPS

อันดับแรกมาถึงก็ดู Volumn ก่อนเลย
lsblk

<img width="569" height="152" alt="image" src="https://github.com/user-attachments/assets/b4dcb711-eca7-4682-8044-8e7766dc0e01" />

Hetzner สร้างชื่อ Volumn น่าเกลียดไปนิด เราต้องเปลี่ยน

sudo ln -s /mnt/HC_Volume_105481441 /mnt/bitcoin

สร้าง Folder ขึ้นมา

sudo mkdir -p /mnt/bitcoin/bitcoind

sudo chown -R 1000:1000 /mnt/bitcoin/bitcoind

จากนั้นก็สร้าง Compose File

ด้วยคำสั่ง sudo nano docker-compose.yml

services:
  bitcoind:
    image: lncm/bitcoind:v27.2
    container_name: bitcoind
    restart: unless-stopped
    volumes:
      - /mnt/bitcoin/bitcoind:/data/.bitcoin
    ports:
      - "8333:8333"
    command:
      -server=1
      -txindex=1
      -rpcuser=bitcoin
      -rpcpassword=strongpassword
      -rpcbind=0.0.0.0
      -rpcallowip=127.0.0.1
      -rpcallowip=172.16.0.0/12
      -rpcport=8332
      -zmqpubrawblock=tcp://0.0.0.0:28332
      -zmqpubrawtx=tcp://0.0.0.0:28333
      -dbcache=1024

จากนั้นให้ทำการเปิด Firewall

sudo ufw allow 8333/tcp   อันนี้สำหรับ Bitcoin P2P

sudo ufw allow 9735/tcp   อันนี้สำหรับ LND ในอนาคต

sudo ufw reload

อันนี้เป็นการทำให้ Docker สามารถใช้งานคำสั่ง docker ได้โดยที่ไม่ต้องมี sudo

sudo usermod -aG docker chanwut

เมื่อเราดู log ก็จะเห็นมันวิ่ง docker logs bitcoind

<img width="852" height="564" alt="image" src="https://github.com/user-attachments/assets/d51f9072-055f-476f-97c5-33db19158962" />

เมื่อคุณทำการ Sync all block เรียบร้อยแล้วจากนั้นให้ทำการติดตั้ง LND ได้เลย

จากนั้นให้คุณทำการสร้าง Docker network ให้ทั้ง 2 Container มาเจอกัน

จากนั้นให้ทำการสร้าง Wallet at LND โดยใช้คำสั่ง docker exec -it lnd lncli create

แล้วก็ตั้งรหัสให้มันซะ จากนั้นก็จด Seed

หลังจากนั้นก็ Unlock Wallet ด้วยคำสั่ง docker exec -it lnd lncli unlock

แล้วก็ดูข้อมูล docker exec -it lnd lncli getinfo

How to change node Alias is here create configure file nano /mnt/bitcoin/lnd/lnd.conf

and put something like

[Application Options]

alias=ChanwutNode

color=#ff9900

and then save

This is how to check information

docker exec -it lnd lncli getnetworkinfo

docker exec -it lnd lncli listpeers

How to connect to other peer with this command docker exec -it lnd lncli connect

docker exec -it lnd lncli connect 0324ba2392e25bff76abd0b1f7e4b53b5f82aa53fddc3419b051b6c801db9e2247@54.244.234.100:20465

And this is how to back up LN node : cp /mnt/bitcoin/lnd/data/chain/bitcoin/mainnet/channel.backup ~/

docker exec -it lnd lncli newaddress p2wkh

docker exec -it lnd lncli walletbalance

docker exec -it lnd lncli openchannel \
--node_key=PUBKEY \
--local_amt=1000000
