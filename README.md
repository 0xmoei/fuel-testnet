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
> 
> Create new wallet and get [Faucet](https://faucet-testnet.fuel.network/)

```console
# Import seed phrase and create password
forc wallet import

# Create account
forc wallet account new

# Check your address
forc wallet accounts

# Deploy Contract , Enter password , Enter 0 , Enter y
forc deploy --testnet

# You get a Contract ID in your Terminal like this, Save it!
Contract counter-contract Deployed!

Network: https://testnet.fuel.network
Contract ID: 0x8342d413de2a678245d9ee39f020795800c7e6a4ac5ff7daae275f533dc05e08
Deployed in block 0x4ea52b6652836c499e44b7e42f7c22d1ed1f03cf90a1d94cd0113b9023dfa636
```

> Congrats, you have completed your first smart contract on Fuel ⛽
> Tweet @fuel_network letting them to know you just built a dapp on Fuel, you might get invited to a private group of builders, be invited to the next Fuel dinner, get alpha on the project, or something 👀.


# Deploy Dapp

<h1 align="center"> Install Dependecies </h1>

```console
# Check Nodejs Version
node --version

# Delete old files
sudo apt-get remove nodejs
sudo apt-get purge nodejs
sudo apt-get autoremove
sudo rm /etc/apt/keyrings/nodesource.gpg
sudo rm /etc/apt/sources.list.d/nodesource.list

# Install Nodejs 18
NODE_MAJOR=18
sudo apt-get update
sudo apt-get install -y ca-certificates curl gnupg
sudo mkdir -p /etc/apt/keyrings

curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | sudo gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg

echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_${NODE_MAJOR}.x nodistro main" | sudo tee /etc/apt/sources.list.d/nodesource.list

sudo apt-get update
sudo apt-get install -y nodejs
node --version
```

<h1 align="center"> Create Dapp Frontend </h1>

```console
cd $HOME && cd fuel-project
npx create-react-app frontend --template typescript
# Success! Created frontend at Fuel/fuel-project/frontend

# Install fuels sdk
ls
cd frontend
npm install fuels @fuels/react @fuels/connectors @tanstack/react-query

# Generate Contract type
npx fuels init --contracts ../counter-contract/ --output ./src/sway-api
npx fuels build

# Building..
Building Sway programs using source 'forc' binary
Generating types..
🎉  Build completed successfully!
```

<h1 align="center"> Modify Dapp Frontend </h1>

```console
# Edit index
nano src/index.tsx

# Delete everything and Paste this code
import React from 'react';
import ReactDOM from 'react-dom/client';
import './index.css';
import App from './App';
import reportWebVitals from './reportWebVitals';
import { FuelProvider } from '@fuels/react';
import {
  defaultConnectors,
} from '@fuels/connectors';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';

const queryClient = new QueryClient();

const root = ReactDOM.createRoot(
  document.getElementById('root') as HTMLElement
);
root.render(
  <React.StrictMode>
    <QueryClientProvider client={queryClient}>
      <FuelProvider
        fuelConfig={{
          connectors: defaultConnectors(),
        }}
      >
        <App />
      </FuelProvider>
    </QueryClientProvider>
  </React.StrictMode>
);

// If you want to start measuring performance in your app, pass a function
// to log results (for example: reportWebVitals(console.log))
// or send to an analytics endpoint. Learn more: https://bit.ly/CRA-vitals
reportWebVitals();
```

> Save ( Ctrl + X , Y , Enter )

```console
# Edit App
nano src/App.tsx

# Delete everything and Paste this code
import { useEffect, useState } from "react";
import {
  useBalance,
  useConnectUI,
  useIsConnected,
  useWallet
} from '@fuels/react';
import { CounterContractAbi__factory  } from "./sway-api"
import type { CounterContractAbi } from "./sway-api";
 
// REPLACE WITH YOUR CONTRACT ID
const CONTRACT_ID = 
  "0x...";
 
export default function Home() {
  const [contract, setContract] = useState<CounterContractAbi>();
  const [counter, setCounter] = useState<number>();
  const { connect, isConnecting } = useConnectUI();
  const { isConnected } = useIsConnected();
  const { wallet } = useWallet();
  const { balance } = useBalance({
    address: wallet?.address.toAddress(),
    assetId: wallet?.provider.getBaseAssetId(),
  });
 
  useEffect(() => {
    async function getInitialCount(){
      if(isConnected && wallet){
        const counterContract = CounterContractAbi__factory.connect(CONTRACT_ID, wallet);
        await getCount(counterContract);
        setContract(counterContract);
      }
    }
    
    getInitialCount();
  }, [isConnected, wallet]);
 
  const getCount = async (counterContract: CounterContractAbi) => {
    try{
      const { value } = await counterContract.functions
      .count()
      .get();
      setCounter(value.toNumber());
    } catch(error) {
      console.error(error);
    }
  }
 
  const onIncrementPressed = async () => {
    if (!contract) {
      return alert("Contract not loaded");
    }
    try {
      await contract.functions
      .increment()
      .call();
      await getCount(contract);
    } catch(error) {
      console.error(error);
    }
  };
 
  return (
    <div style={styles.root}>
      <div style={styles.container}>
        {isConnected ? (
          <>
            <h3 style={styles.label}>Counter</h3>
            <div style={styles.counter}>
              {counter ?? 0}
            </div>
 
            {balance && balance.toNumber() === 0 ? (
              <p>Get testnet funds from the <a target="_blank" rel="noopener noreferrer"  href={`https://faucet-testnet.fuel.network/?address=${wallet?.address.toAddress()}`}>Fuel Faucet</a> to increment the counter.</p>
          ) : 
          (
            <button
            onClick={onIncrementPressed}
            style={styles.button}
            >
              Increment Counter
            </button>
          )
          }
            
          <p>Your Fuel Wallet address is:</p>
          <p>{wallet?.address.toAddress()}</p>
          </>
        ) : (
          <button
          onClick={() => {
            connect();
          }}
          style={styles.button}
          >
            {isConnecting ? 'Connecting' : 'Connect'}
          </button>
        )}
      </div>
    </div>
  );
}
 
const styles = {
  root: {
    display: 'grid',
    placeItems: 'center',
    height: '100vh',
    width: '100vw',
    backgroundColor: "black",
  } as React.CSSProperties,
  container: {
    color: "#ffffffec",
    display: "flex",
    flexDirection: "column",
    alignItems: "center",
  } as React.CSSProperties,
  label: {
    fontSize: "28px",
  },
  counter: {
    color: "#a0a0a0",
    fontSize: "48px",
  },
  button: {
    borderRadius: "8px",
    margin: "24px 0px",
    backgroundColor: "#707070",
    fontSize: "16px",
    color: "#ffffffec",
    border: "none",
    outline: "none",
    height: "60px",
    padding: "0 1rem",
    cursor: "pointer"
  },
}
```

<h1 align="center"> Run Dapp </h1>

```console
# Start Dapp
npm start

Compiled successfully!

You can now view frontend in the browser.

  Local:            http://localhost:3000
  On Your Network:  http://192.168.4.48:3000

Note that the development build is not optimized.
To create a production build, use npm run build.
```

> Click the "Connect" button and select the wallet you have installed to connect your wallet.
> 
> Once connected, if there are no funds in your wallet, you will see a link to get testnet funds.
>
> If you have testnet ETH on Fuel, you should see the counter value and increment button:
![quickstart-frontend](https://github.com/0xmoei/fuel-testnet/assets/90371338/50a7d737-42f8-44f9-b403-a9ff6a6ec416)

> You just built a fullstack dapp on Fuel! ⛽
>
> Tweet @fuel_network letting them know you just built a dapp on Fuel, you might get invited to a private group of builders, be invited to the next Fuel dinner, get alpha on the project, or something 👀.


## Deploy testnet Node

<h1 align="center"> Ignore this if you already installed Dependecies and Fuel before </h1>

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
