Once you have navigated to https://github.com/IntersectMBO/cardano-node/releases/tag/9.1.0 and downloaded the appropriate binary under Assets , please follow the below steps to sync to the preview testnet:

### Download all the config files from the  into ~/testnet/config/preview

### FOLDER ORGANIZATION
- node files extracted under $HOME/cardano-node-9.1.0-macos
- testnet folder as in  ~/testnet (refer environment variable below)
- preview network configuration files under $HOME/cardano-node-9.1.0-macos/share/preview/ 
- rename /share/preview/config.json to config.yaml



UPDATE your .bashrc / .zshrc file accordingly AND RELOAD IT AFTER SAVING IT by either restarting the terminal or running the "source ~/.bashrc" (or .zshrc) file!


## FOR MacOS
```
export CARDANO_NODE="$HOME/cardano-node-9.1.0-macos"

export PATH="$CARDANO_NODE:$PATH"
export TESTNETPATH="$HOME/testnet"

#if there is a space, zsh breaks unless you expand the variable using parentheses
#magic id for preview network is 2, preprod 1
export TESTNET=(--testnet-magic 2)

#socket path remains same, no change required as we're creating this file ourselves
export CARDANO_NODE_SOCKET_PATH="$TESTNETPATH/node.socket"

alias previewnode='$CARDANO_NODE/bin/cardano-node run --topology $CARDANO_NODE/share/preview/topology.json --database-path $TESTNETPATH/db/preview/ --socket-path $CARDANO_NODE_SOCKET_PATH --port 3001 --config $CARDANO_NODE_/share/preview/config.yaml'


alias ctip='cardano-cli query tip $TESTNET'

function utxo() { cardano-cli query utxo $TESTNET --address $1 ; }
function utxof() { cardano-cli query utxo $TESTNET --address $(cat $1) ; }
function submit() { cardano-cli transaction submit --tx-file $1 $TESTNET ;}

```

If you get an error when you run cardano-cli or cardano-node about libsodium.23.dylib, we will download, compile and install `libsodium`.

```bash
git clone https://github.com/input-output-hk/libsodium
cd libsodium
git checkout 66f017f1
./autogen.sh
./configure
make
sudo make install
```

Then we will add the following environment variables to your shell profile. E.G `$HOME/.zshrc` or `$HOME/.bashrc` depending on what shell application you are using. Add the following to the bottom of your shell profile/config file so the compiler can be aware that `libsodium` is installed on your system.

```bash
export LD_LIBRARY_PATH="/usr/local/lib:$LD_LIBRARY_PATH"
export PKG_CONFIG_PATH="/usr/local/lib/pkgconfig:$PKG_CONFIG_PATH"
```


## Installation complete! Now to run the cardano-node, just 
## restart the terminal (or run source ~/.bashrc (or .zshrc) 



## To start the cardano-node in preview network , type the command below and hit ENTER
```bash
previewnode
```

when you run the binaries, if you get a "App Can’t Be Opened Because It Is From an Unidentified Developer" popup, refer: https://www.lifewire.com/fix-developer-cannot-be-verified-error-5183898

Keep clicking "Allow Anyway" in the security prompt in settings for all files which give a popup

 Alternatively you can use the below commands to temporarily disable the gatekeeper security. 
WARNING:  disables the system policies entirely! Perform at your own risk! 
To enable to allow apps from anywhere to be installed use the following command in terminal ::
```bash
sudo spctl --master-disable
```
To re-enable system policies, use the following command
```bash
sudo spctl --master-enable
```

you can always type 
```
spctl --status
```
if it's still enabled you'll get assessments enabled in response.


## How to deal with common errors.
### if after an improper shut-down, you get the below error

```
cardano-node: The db is used by another process. File "/home/bharat/testnet/db/preview/lock" is locked.
```

then either restart the system or kill the cardano-node process (not recommended!)

```bash
sudo killall cardano-node
```

### After this kind of an improper shutdown, you might need to delete all the files in the /testnet/db/preview or /testnet/db/preprod folder and resync the database 

### NOTE: use the rm -rf command with care as if it is run in the wrong directory, it can delete important files
### Ensure that you only run it within the testnet/db/.. folder
```bash
cd $TESTNET/db/preview
rm -rf ./* 
rm -rf ./.*
```

Then re-launch the node and it will quickly sync to the latest block (takes only a minute or two right now!)



## To get some funds from the Ada faucet (WORKING AS OF NOW)
Use this faucet to get preview / preprod ADA: https://docs.cardano.org/cardano-testnet/tools/faucet
