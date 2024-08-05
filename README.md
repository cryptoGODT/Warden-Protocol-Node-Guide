# Warden-Protocol-Node-Guide

![Warden-Protocol](https://github.com/user-attachments/assets/3382e004-1170-4917-84ae-47d07627d499)


# Running a Validator Node on Buenavista 🌐🛠️

This guide provides a step-by-step process for setting up and running a validator node on the Buenavista network. Follow these instructions to get started! 📜🚀

## Version History 📅

| Release | Upgrade Block Height | Upgrade Date      |
|---------|-----------------------|-------------------|
| v0.3.0  | genesis               |                   |
| v0.4.0  | 1,520,000             | August 1, 2024    |

## Prerequisites 🖥️

For optimal performance, we recommend running public testnet nodes with the following specifications:

- **CPU:** At least 8 cores 🧠
- **RAM:** 32GB 📈
- **Disk Space:** 300GB 💽

You will also need to install Go. 🌟

## 1. Install Wardend 📦

### Option 1: Use the Prebuilt Binary

1. **Download the Binary** 🗂️

   Download the binary for your platform from the [release page](https://github.com/warden-protocol/wardenprotocol/releases). Unzip the archive; it contains the `wardend` binary.

2. **Initialize the Chain Home Folder** 🌐

   ```bash
   ./wardend init <custom_moniker>
   ```

### Option 2: Use the Source Code

1. **Build the Binary** ⚙️

   ```bash
   git clone --depth 1 --branch v0.3.0 https://github.com/warden-protocol/wardenprotocol/
   cd wardenprotocol
   just build
   ```

2. **Initialize the Chain Home Folder** 🌐

   ```bash
   build/wardend init <custom_moniker>
   ```

## 2. Configure Wardend ⚙️

1. **Prepare the Genesis File** 🌟

   ```bash
   cd $HOME/.warden/config
   rm genesis.json
   wget https://raw.githubusercontent.com/warden-protocol/networks/main/testnets/buenavista/genesis.json
   ```

2. **Set the Mandatory Configuration Options** 🔧

   Update the `app.toml` and `config.toml` files:

   ```bash
   # Set minimum gas price
   sed -i 's/minimum-gas-prices = ""/minimum-gas-prices = "0.0025uward"/' app.toml

   # Set persistent peers
   sed -i 's/persistent_peers = ""/persistent_peers = "92ba004ac4bcd5afbd46bc494ec906579d1f5c1d@52.30.124.80:26656,ed5781ea586d802b580fdc3515d75026262f4b9d@54.171.21.98:26656"/' config.toml
   ```

## 3. Set Up State Sync ⏳

This step is optional but recommended to speed up the initial sync. It allows you to download the state from a trusted node and then only download new blocks from the network. 🚀

1. **Get Trusted RPC Endpoint** 🌐

   Use the following RPC endpoint:

   ```bash
   export SNAP_RPC_SERVERS="https://rpc.buenavista.wardenprotocol.org:443,https://rpc.buenavista.wardenprotocol.org:443"
   ```

2. **Retrieve Trusted Block Height and Hash** 🔍

   ```bash
   export LATEST_HEIGHT=$(curl -s "https://rpc.buenavista.wardenprotocol.org/block" | jq -r .result.block.header.height)
   export BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000))
   export TRUST_HASH=$(curl -s "https://rpc.buenavista.wardenprotocol.org/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

   # Check if variables are set correctly
   echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH
   ```

3. **Add State Sync Configuration** ⚙️

   ```bash
   sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
   s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC_SERVERS\"| ; \
   s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
   s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" $HOME/.warden/config/config.toml
   ```

## 4. Start the Node 🚀

Now you can start your node with the following command:

```bash
wardend start
```

Your node will connect to the persistent peers and begin downloading blocks. You can check the logs to monitor progress. 📈📝
