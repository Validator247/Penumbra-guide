# Penumbra-guide

First make sure you have installed the latest system components

# System requirements

    8GB RAM
    2-4 vCPUS
    ~200GB persistent storage (~20GB/week)

# Installing pd 

    curl -sSfL -O https://github.com/penumbra-zone/penumbra/releases/download/v0.75.0/pd-x86_64-unknown-linux-gnu.tar.gz
    tar -xf pd-x86_64-unknown-linux-gnu.tar.gz
    sudo mv pd-x86_64-unknown-linux-gnu/pd /usr/local/bin/

    # confirm the pd binary is installed by running:
    pd --version

# Installing CometBFT    

From Source

    echo export GOPATH=\"\$HOME/go\" >> ~/.bash_profile
    echo export PATH=\"\$PATH:\$GOPATH/bin\" >> ~/.bash_profile

Get Source Code

    git clone https://github.com/cometbft/cometbft.git
    cd cometbft
    git checkout v0.37.5
    make install

Compile

    make install

to put the binary in $GOPATH/bin or use:

    make build

check version

    cometbft version

# Joining a Testnet   

Resetting state

    pd testnet unsafe-reset-all

Generating configs

    pd testnet join --external-address IP_ADDRESS:26656 --moniker MY_NODE_NAME

Running pd and cometbft

    pd start

Set up SystemD, you can create a separate tmux for convenient operation. After creating, save.

    nano /etc/systemd/system/pd.service

edit service files to customize for your system


    [Unit]
    Description=pd node
    After=network-online.target
    [Service]
    User=root
    WorkingDirectory=/root/.penumbra/testnet_data/node0
    ExecStart=/usr/local/bin/pd start
    Restart=on-failure
    RestartSec=5
    LimitNOFILE=65535
    [Install]
    WantedBy=multi-user.target
    
    sudo systemctl daemon-reload
    sudo systemctl restart penumbra cometbft


Then (perhaps in another terminal), run CometBFT, specifying --home:

    cometbft start --home ~/.penumbra/testnet_data/node0/cometbft

Check logs

    sudo journalctl -u pd -f

# Creat Wallet 

Installing pcli

    curl --proto '=https' --tlsv1.2 -LsSf https://github.com/penumbra-zone/penumbra/releases/download/v0.75.0/pcli-installer.sh | sh

Create PATH Environment Variable

    export PATH="$HOME/.cargo/bin:$PATH"
    source ~/.bashrc

confirm the pcli binary is installed by running:

    pcli --version

Generating a Wallet

    pcli init soft-kms generate

# Becoming a validator

Create a Definition Template

    pcli validator definition template \
    --tendermint-validator-keyfile ~/.penumbra/testnet_data/node0/cometbft/config/priv_validator_key.json \
    --file validator.toml

Creat config validator: Edit parameters and save

    nano validator.toml

Uploading a definition

    pcli validator definition upload --file validator.toml

And verify that it’s known to the chain:

    pcli query validator list --show-inactive

# Delegating to your validator

First find your validator’s identity key:  

    pcli validator identity

Delegate 

    pcli tx delegate 1penumbra --to [YOUR_VALIDATOR_IDENTITY_KEY]

check balance

    pcli view balance

Updating your validator
First fetch your existing validator definition from the chain:

    pcli validator definition fetch --file validator.toml

After updating the validator definition you can upload it again to update your validator metadata on-chain:

    pcli validator definition upload --file validator.toml

Read details from the team's instructions

    https://guide.penumbra.zone/node/pd/validator.html

# DONE !        
        
