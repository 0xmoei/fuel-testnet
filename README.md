# Deploy Contract, Dapp & Full Node on Fuel Network Testnet

# Deploy Contract

<h1 align="center"> Dependecies </h1>

```console
# Install Dependecies
sudo apt-get update && apt-get upgrade -y
sudo apt-get install git-all -y
sudo apt-get install curl screen -y

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
fuelup toolchain install latest
fuelup self update
fuelup update
fuelup default latest

# Check version
fuelup --version
```

<h1 align="center"> Create project </h1>

```console
# Create project
mkdir fuel-project
cd fuel-project
forc new counter-contract

# Edit Contract
nano counter-contract/src/main.sw

# Delete everything and Paste this code
contract;
 
storage {
    counter: u64 = 0,
}
 
abi Counter {
    #[storage(read, write)]
    fn increment();
 
    #[storage(read)]
    fn count() -> u64;
}
 
impl Counter for Contract {
    #[storage(read)]
    fn count() -> u64 {
        storage.counter.read()
    }
 
    #[storage(read, write)]
    fn increment() {
        let incremented = storage.counter.read() + 1;
        storage.counter.write(incremented);
    }
}

# Build Contract
cd counter-contract
forc build
```

<h1 align="center"> Deploy Contract </h1>

> First, Download [Fuel-Wallet](https://wallet.fuel.network/docs/install/)
> Create new wallet and get [Faucet](https://faucet-testnet.fuel.network/)

```console
# Import seed phrase and create password
forc wallet import

# Create account
forc wallet account new

# Check your address
forc wallet accounts
```



## Deploy testnet Node

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

# Deploy Contract
forc deploy --testnet

# You get a Contract ID in your Terminal like this, Save it!
Contract counter-contract Deployed!

Network: https://testnet.fuel.network
Contract ID: 0x8342d413de2a678245d9ee39f020795800c7e6a4ac5ff7daae275f533dc05e08
Deployed in block 0x4ea52b6652836c499e44b7e42f7c22d1ed1f03cf90a1d94cd0113b9023dfa636
```

> Congrats, you have completed your first smart contract on Fuel ⛽
> Tweet @fuel_network letting them to know you just built a dapp on Fuel, you might get invited to a private group of builders, be invited to the next Fuel dinner, get alpha on the project, or something 👀.



<h1 align="center"> Getting an Ethereum Sepolia API Key </h1>

> Get Ethereum Sepolia API in [Alchemy](https://dashboard.alchemy.com/)
![Screenshot_5](https://github.com/0xmoei/Fuel-testnet/assets/90371338/ddb4cd12-a05a-4bb5-9509-e9481159ee56)


<h1 align="center"> Generate a P2P Key </h1>

```console
fuel-core-keygen new --key-type peering
# Make sure you save peerid and secret
```

<h1 align="center"> Chain Config </h1>

```console
# Clone the repository
git clone https://github.com/0xmoei/fuel-testnet

# Go to testnet directory
cd fuel-testnet/testnet-node
```

<h1 align="center"> Run Node </h1>

```console
# Start Screen
screen -S fuel

# Replace NAME , P2P_Secret , ETH_SEPOLIA_API
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

# Minimeze Node
Screen + A + D

# Return Node
Screen -r fuel
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
