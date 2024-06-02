# Fuel-testnet

<h1 align="center"> Dependecies </h1>

```console
# Install Dependecies
sudo apt-get update && apt-get upgrade -y
sudo apt install curl
sudo apt install screen

# Install Rust
curl --proto '=https' --tlsv1.3 https://sh.rustup.rs -sSf | sh
source "$HOME/.cargo/env"
rustup install stable
rustup update stable
rustup default stable
```

<h1 align="center"> Install Fuel Toolchain </h1>

```console
# Install Fuel
curl https://install.fuel.network | sh
# Press y then Enter

# Set Path
source /root/.bashrc

# Set fuelup testnet
fuelup toolchain install testnet
fuelup default testnet

# Check version
fuelup --version
```

<h1 align="center"> Getting an Ethereum Sepolia API Key </h1>

> Get Ethereum Sepolia API in [Alchemy](https://dashboard.alchemy.com/)
![Screenshot_5](https://github.com/0xmoei/Fuel-testnet/assets/90371338/ddb4cd12-a05a-4bb5-9509-e9481159ee56)


<h1 align="center"> Generate a P2P Key </h1>

```console
fuel-core-keygen new --key-type peering
### Make sure you save this somewhere safe so you don't need to generate a new key pair in the future. ###
```

<h1 align="center"> Chain Config </h1>

```console
# Clone the repository
git clone https://github.com/0xmoei/fuel-testnet

# Go to testnet directory
cd fuel-testnet/testnet-node
```

<h1 align="center"> Run Node </h1>
> Replace NAME , P2P_Secret , ETH_SEPOLIA_API

```console
fuel-core run \
--service-name NNAME \
--keypair P2P_Secret \
--relayer ETH_SEPOLIA_API \
--ip 0.0.0.0 --port 4000 --peering-port 30333 \
--db-path  ~/.testnet \
--snapshot $HOME/fuel-testnet/testnet-node \
--utxo-validation --poa-instant false --enable-p2p \
--min-gas-price 1 --max-block-size 18874368  --max-transmit-size 18874368 \
--reserved-nodes /dns4/p2p-devnet.fuel.network/tcp/30333/p2p/16Uiu2HAm6pmJUedRFjennk4A8yWL6zCApHCuykzRRroqMjjxZ8o6,/dns4/p2p-devnet.fuel.network/tcp/30334/p2p/16Uiu2HAm8dBwTRzqazCMqQDdR8thMa7BKiW4ep2B4DoQQp6Qhyfd  \
--sync-header-batch-size 100 \
--enable-relayer \
--relayer-v2-listening-contracts 0x01855B78C1f8868DE70e84507ec735983bf262dA \
--relayer-da-deploy-height 5827607 \
--relayer-log-page-size 2000
```

> Now wait until you fully download Logs for Blocks

<h1 align="center"> Connecting to the node from Wallet </h1>

```
# Open port
sudo ufw allow 4000
```

> You can now add this rpc to your wallet networks
```
http://your_ip:4000/graphql
```
