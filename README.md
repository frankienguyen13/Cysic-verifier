
# Instructions for running multiple verifiers

This is a guide to running multiple verifiers on 1 device by using PM2, make sure your hardware is capable of running large numbers. (The project recommends around 6-8Gb of RAM and 1 core CPU for 1 verifier worker).



## Install 
Install dependencies
```bash
  sudo apt-get update && sudo apt install nano && sudo apt install -y jq && sudo apt install -y npm && sudo npm install -g pm2
```
Enter wallets informations into `cysic.txt` file
```bash
nano cysic.txt
```
Format `wallet,evm` each line 1 wallet

![App Screenshot](https://github.com/frankienguyen13/Cysic-verifier/blob/main/wallets.PNG?raw=true)

Create `setup_cysic.sh` with the following content
```bash
#!/bin/bash
# File containing the list of wallets and claim addresses
wget https://github.com/cysic-labs/cysic-phase3/releases/download/v1.0.0/verifier_linux && mv verifier_linux verifier
WALLET_FILE="cysic.txt"
# Read the wallet file line by line
while IFS=, read -r WALLET_NAME CLAIM_REWARD_ADDRESS
do
  # Create a new directory for each wallet
  WALLET_DIR=~/cysic-verifier-$WALLET_NAME
  rm -rf $WALLET_DIR
  mkdir $WALLET_DIR
  # Download necessary files into the wallet directory
  cp verifier $WALLET_DIR/verifier
  #cp libdarwin_verifier.so $WALLET_DIR/libdarwin_verifier.so
  #cp librsp.so $WALLET_DIR/librsp.so

  # Create the configuration file
  cat <<EOF > $WALLET_DIR/config.yaml
# Not Change
chain:
  # Not Change
  # endpoint: "node-pre.prover.xyz:80"
  endpoint: "grpc-testnet.prover.xyz:80"
  # Not Change
  chain_id: "cysicmint_9001-1"
  # Not Change
  gas_coin: "CYS"
  # Not Change
  gas_price: 10
  # Modify Here：! Your Address (EVM) submitted to claim rewards
claim_reward_address: "$CLAIM_REWARD_ADDRESS"
server:
  # don't modify this
  # cysic_endpoint: "https://api-pre.prover.xyz"
  cysic_endpoint: "https://ws-pre.prover.xyz"
  verify_endpoint: "http://verifier-rpc.prover.xyz:50052"
EOF
  # Set executable permissions and create the start script
  chmod +x $WALLET_DIR/verifier
  echo "LD_LIBRARY_PATH=. CHAIN_ID=534352 ./verifier" > $WALLET_DIR/start.sh
  chmod +x $WALLET_DIR/start.sh

done < "$WALLET_FILE"
```
Create `run_pm2.sh`
```bash
#!/bin/bash

# Path to the cysic.txt file
input_file="cysic.txt"

# Set the CHAIN_ID to 534352 for all wallets
CHAIN_ID=534352

# Read each line of cysic.txt
while IFS=, read -r folder_name _; do
  # Construct the folder path
  folder_path="$HOME/cysic-verifier-$folder_name"

  # Check if the folder exists
  if [ -d "$folder_path" ]; then
    echo "Starting verifier in $folder_path with PM2 as $folder_name"

    # Use a script file to start the process properly
    pm2 start bash --name "$folder_name" -- -c "cd $folder_path && LD_LIBRARY_PATH=. CHAIN_ID=$CHAIN_ID ./verifier"

  else
    echo "Folder $folder_path does not exist. Skipping..."
  fi
done < "$input_file"
```
Set executable permission and run setup file
```bash
chmod +x setup_cysic.sh
./setup_cysic.sh
```
This executable file will create folders cysic-verifier-wallet1,cysic-verifier-wallet2.. corresponding to the information in cysic.txt

Run `run_pm2.sh` file
```bash
chmod +x run_pm2.sh
./run_pm2.sh
```

    
## Screenshots

![App Screenshot](https://github.com/frankienguyen13/Christmas/blob/master/images/pm2list.PNG?raw=true)

## Check logs
To check log of 1 verifier use `pm2 log 0` (0 is id of process) or `pm2 log wallet1`

To get list of all running verifiers
`pm2 list`

To check submitted txHash of verifier (change `wallet1` value with your verifier name)
```bash
grep --text 'txHash' /root/.pm2/logs/wallet1-error.log
```


