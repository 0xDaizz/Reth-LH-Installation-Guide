# Reth-LH-Installation-Guide
How to Build Reth + Lighthouse + Prometheus + Grafana

thanks to : [teebaumcrypto](https://gist.github.com/teebaumcrypto/5c7a30ae9f25d3f628100188149b1fb1)

## 0. Spec & Env.
- CPU : AMD Ryzen 9 7900X (with Eco mode 65W)
- SSD : WD SN850X 4TB
- RAM : Patriot DDR5 32 * 2 = 64GB

- OS : WSL - Ubuntu 22.04 LTS in Windows 10

> **Note**
> 
> If you want to run reth on WSL2, make sure you have expanded your virtual hard disk size before start syncing reth.


## 0-1. Sync time
![image](https://github.com/0xDaizz/Reth-LH-Installation-Guide/assets/90135051/1a9c7a67-5392-4bb5-805c-fe5c16c168c6)

About 6~7 days

---
## 1. Prometheus & Grafana

### 1-1. Prometheus
- installation
```
sudo apt update
sudo apt install prometheus
```

- edit prometheus.yml
Directory: ```/etc/prometheus/prometheus.yml```

Replace .yml with [this](https://github.com/paradigmxyz/reth/blob/main/etc/prometheus/prometheus.yml)

```
download the file, and then


sudo cp /mnt/c/users/<username>/downloads/prometheus.yml /etc/prometheus/prometheus.yml
```

- run
```
sudo systemctl start prometheus

# automatically run on log-on
sudo systemctl enable prometheus
```

### 1-2. Grafana
- installation
ref: [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-install-and-secure-grafana-on-ubuntu-22-04)
```
wget -q -O - https://packages.grafana.com/gpg.key | gpg --dearmor | sudo tee /usr/share/keyrings/grafana.gpg > /dev/null

echo "deb [signed-by=/usr/share/keyrings/grafana.gpg] https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list

sudo apt update
sudo apt install grafana
sudo systemctl start grafana-server

# automatically run on log-on
sudo systemctl enable grafana-server
```

- add dashboard
1. enter http://localhost:3000 - log in admin/admin
2. left toggle menu button - administration - Data Sources - Add new data source - Prometheus
3. Fill HTTP - Prometheus server URL with http://localhost:9090
4. click Save & test
5. left toggle menu button - Dashboards - New - Import
6. Upload JSON file from [here](https://github.com/paradigmxyz/reth/blob/main/etc/grafana/dashboards/overview.json)
7. click Load

---

## 2. Build Reth and Lighthouse

- Make sure you have already installed Rust!
- if not : ```curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh```

### 2-1. Reth Build

```
# Start from Home directory
cd ~

git clone https://github.com/paradigmxyz/reth
cd reth

# Choose the version of Reth
git checkout v0.1.0-alpha.4

# Build
RUSTFLAGS="-C target-cpu=native" cargo build --profile maxperf --features jemalloc

# Cf. Build on Windows:
$env:RUSTFLAGS="-C target-cpu=native"; cargo build --profile maxperf


```

### 2-2. Lighthouse Build
```
# Start from Home directory
cd ~

# dependencies
sudo apt install -y git gcc g++ make cmake pkg-config llvm-dev libclang-dev clang protobuf-compiler

git clone https://github.com/sigp/lighthouse.git
cd lighthouse
git checkout stable
PROFILE=maxperf make

# Cf. Build on Windows
$env:PROFILE = "maxperf"; make

```

### 2-3 Run Reth & LH
- You may customize the command flags as you want.
  
- **Make sure you run LH first!! (because of the generation of jwt file)**

Lighthouse
```
lighthouse bn  --network mainnet --execution-endpoint http://localhost:8551 --execution-jwt ~/data/jwt.hex --checkpoint-sync-url https://mainnet.checkpoint.sigp.io --disable-deposit-contract-sync --datadir ~/data/lighthouse_data
```

Reth
```
RUST_LOG=info ~/reth/target/maxperf/reth node --datadir ~/data/reth_data --authrpc.jwtsecret ~/data/jwt.hex --ws --ws.addr="127.0.0.1" --ws.api=eth,web3,net,txpool --http --http.api=eth,web3,net,txpool --http.addr="127.0.0.1" --http.port=8545 --metrics 127.0.0.1:9001


**If you wanna run on WSL and use RPC endpoints on your local computer:**

RUST_LOG=info ~/reth/target/maxperf/reth node --datadir ~/data/reth_data --authrpc.jwtsecret ~/data/jwt.hex --ws --ws.addr="0.0.0.0" --ws.port=8545 --ws.api=eth,web3,net,txpool --http --http.api=eth,web3,net,txpool --http.addr="0.0.0.0" --http.port=8545 --metrics 127.0.0.1:9001
```

**on Windows:**

Lighthouse
```
lighthouse bn --network mainnet --execution-endpoint http://localhost:8551 --execution-jwt ..\data\jwt.hex --checkpoint-sync-url https://mainnet.checkpoint.sigp.io --disable-deposit-contract-sync --datadir ..\data\lighthouse_data
```

Reth
```
$env:RUST_LOG = "info"; ./target/maxperf/reth node --datadir ../data/reth_data --authrpc.jwtsecret ../data/jwt.hex --ws --ws.addr="127.0.0.1" --ws.api=eth,web3,net,txpool --http --http.api=eth,web3,net,txpool --http.addr="127.0.0.1" --http.port=8545 --metrics 127.0.0.1:9001
```
 
Done!
