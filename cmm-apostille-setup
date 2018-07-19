#/bin/bash

readonly RELEASE=https://github.com/CommerciumBlockchain/Commercium-TESTNET/releases/download/TESTNET-0.16.0/commercium-TESTNET-0.16.0-bin-linux.tar.gz
readonly FILENAME=commercium-TESTNET-0.16.0-bin-linux.tar.gz
readonly DIR=commercium-TESTNET-0.16.0-bin-linux

clear
cd ~ 
echo "12345678901234567890123456789012345678901234567890123456789012345678901234567890"
echo "********************************************************************************"
echo "*      Ubuntu 16.04 x64 is the recommended opearting system for this install.  *"
echo "*                                                                              *"
echo "*      This script will install and configure your Commercium Apostille.       *"
echo "********************************************************************************"
echo && echo && echo
echo "********************************************************************************"
echo "* COLD WALLET PART 1 - DO THIS ON YOUR FULL NODE DESKTOP WALLET                *"
echo "********************************************************************************"
echo "* 1. Open your wallet on your desktop                                          *"
echo "* 2. Go to menu Tools -> Debug Console                                         *"
echo "* 3. Run the following command: masternode genkey                              *"
echo "*    You should see a long string that starts with 5,                          *"
echo "*    5xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx                           *"
echo "*    This is your MASTERNODE PRIVATE KEY                                       *"  
echo "* 4. Keep this key safe, note it down, enter it in the next step               *"
echo "* 5. Send to your own address an amount of exact 20000 in a single transaction *"
echo "*    (testnet)                an amount of exact  1000 in a single transaction *"
echo "********************************************************************************"
echo && echo && echo
sleep 3

# Check for systemd
systemctl --version >/dev/null 2>&1 || { echo "systemd is required. Are you using Ubuntu 16.04?"  >&2; exit 1; }

if [ '${uname -m}' -ne 'x86_64' ]; then
	echo "systemd is required. Are you using Ubuntu 16.04 64bit?"  >&2; exit 1;
fi

# Gather input from user
read -e -p "Masternode Private Key (e.g. 5edfjLCUzGczZi3JQw8GHp434R9kNY33eFyMGeKRymkB56G4324h) : " key
if [[ "$key" == "" ]]; then
    echo "WARNING: No private key entered, exiting!!!"
    echo && exit
fi
read -e -p "Server IP Address : " ip
echo && echo "Pressing ENTER will use the default value for the next prompts."
echo && sleep 3
read -e -p "Add swap space? (Recommended) [Y/n] : " add_swap
if [[ ("$add_swap" == "y" || "$add_swap" == "Y" || "$add_swap" == "") ]]; then
    read -e -p "Swap Size [4G] : " swap_size
    if [[ "$swap_size" == "" ]]; then
        swap_size="4G"
    fi
fi    
read -e -p "Install Fail2ban? (Recommended) [Y/n] : " install_fail2ban
read -e -p "Install UFW and configure ports? (Recommended) [Y/n] : " UFW

# Add swap if needed
if [[ ("$add_swap" == "y" || "$add_swap" == "Y" || "$add_swap" == "") ]]; then
    if [ ! -f /swapfile ]; then
        echo && echo "Adding swap space..."
        sleep 3
        sudo fallocate -l $swap_size /swapfile
        sudo chmod 600 /swapfile
        sudo mkswap /swapfile
        sudo swapon /swapfile
        echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
        sudo sysctl vm.swappiness=10
        sudo sysctl vm.vfs_cache_pressure=50
        echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf
        echo 'vm.vfs_cache_pressure=50' | sudo tee -a /etc/sysctl.conf
    else
        echo && echo "WARNING: Swap file detected, skipping add swap!"
        sleep 3
    fi
fi


# Add masternode group and user
sudo groupadd apostille
sudo useradd -m -g apostille apostille

# Update system 
echo && echo "Upgrading system..."
sleep 3
sudo apt-get -y update

# Install fail2ban if needed
if [[ ("$install_fail2ban" == "y" || "$install_fail2ban" == "Y" || "$install_fail2ban" == "") ]]; then
    echo && echo "Installing fail2ban..."
    sleep 3
    sudo apt-get -y install fail2ban
    sudo service fail2ban restart 
fi

# Install firewall if needed
if [[ ("$UFW" == "y" || "$UFW" == "Y" || "$UFW" == "") ]]; then
    echo && echo "Installing UFW..."
    sleep 3
    sudo apt-get -y install ufw
    echo && echo "Configuring UFW..."
    sleep 3
    sudo ufw default deny incoming
    sudo ufw default allow outgoing
    sudo ufw allow ssh
    sudo ufw allow 12018/tcp
    echo "y" | sudo ufw enable
    echo && echo "Firewall installed and enabled!"
fi

# Download Commercium and install
echo && echo "Downloading Commercium..."
sleep 3
wget $RELEASE
tar -xzvf $FILENAME
cd $DIR
sudo chmod +x commerciumd
sudo chmod +x commercium-cli 
sudo add-apt-repository -y ppa:bitcoin/bitcoin
sudo apt-get update
sudo apt-get install build-essential libtool autotools-dev automake pkg-config libssl-dev libevent-dev bsdmainutils -y
sudo apt-get install libboost-system-dev libboost-filesystem-dev libboost-chrono-dev libboost-program-options-dev libboost-test-dev libboost-thread-dev -y
sudo apt-get update
sudo apt-get install libdb4.8-dev libdb4.8++-dev -y 
sudo apt-get install libminiupnpc-dev -y
sudo apt-get install libzmq3-dev -y
sudo apt-get install libqrencode-dev -y
sudo cp commercium{d,-cli} /usr/local/bin


# Create config for commerciumd
echo && echo "Configuring commerciumd..."
sleep 3
rpcuser=`cat /dev/urandom | tr -dc 'a-zA-Z0-9' | fold -w 32 | head -n 1`
rpcpassword=`cat /dev/urandom | tr -dc 'a-zA-Z0-9' | fold -w 32 | head -n 1`
sudo mkdir -p /home/apostille/.commerciumcore
sudo touch /home/apostille/.commerciumcore/commercium.conf
echo '
rpcuser='$rpcuser'
rpcpassword='$rpcpassword'
rpcallowip=127.0.0.1
listen=1
server=1
daemon=0 # required for systemd
txindex=1
externalip='$ip'
masternodeprivkey='$key'
masternode=1
' | sudo -E tee /home/apostille/.commerciumcore/commercium.conf
sudo chown -R apostille:apostille /home/apostille/.commerciumcore


# Setup systemd service
echo && echo "Starting Commercium Daemon..."
sleep 3
sudo touch /etc/systemd/system/commerciumd.service
echo '[Unit]
Description=commerciumd
After=network.target

[Service]
Type=simple
User=apostille
WorkingDirectory=/home/apostille
ExecStart=/usr/local/bin/commerciumd -conf=/home/apostille/.commerciumcore/commercium.conf -datadir=/home/apostille/.commerciumcore
ExecStop=/usr/local/bin/commercium-cli -conf=/home/apostille/.commerciumcore/commercium.conf -datadir=/home/apostille/.commerciumcore stop
Restart=on-abort

[Install]
WantedBy=multi-user.target
' | sudo -E tee /etc/systemd/system/commerciumd.service
sudo systemctl enable commerciumd
sudo systemctl start commerciumd


# Add alias to run commercium-cli
echo && echo "Masternode setup complete!"
touch ~/.bash_aliases
echo "alias commercium-cli='commercium-cli -conf=/home/apostille/.commerciumcore/commercium.conf -datadir=/home/apostille/.commerciumcore'" | tee -a ~/.bash_aliases
alias commercium-cli='commercium-cli -conf=/home/apostille/.commerciumcore/commercium.conf -datadir=/home/apostille/.commerciumcore'

echo && echo
echo "********************************************************************************"
echo "* COLD WALLET PART 2 - DO THIS ON YOUR FULL NODE DESKTOP WALLET                *"
echo "********************************************************************************"
echo "* 1. Open your wallet on your desktop                                          *"
echo "* 2. Go to menu Tools -> Debug Console                                         *"
echo "* 3. Run the following command: masternode outputs                             *"
echo "*    You should see output like the following if you have a transaction with   *"
echo "*    exactly 20000 CMM:                                                        *"
echo "*    {\"12345678xxxxxxxxxxxxxxx\": \"0\"}                                          *"  
echo "*                                                                              *"
echo "*    The value on the left is your txid and the right is the vout              *"
echo "* 3. Go to menu Settings -> Options -> check the [Show Masternode Tab]         *"
echo "* 4. Go to menu Tools -> Open Masternode Configuration File                    *"
echo "*    Add a line to the bottom of the file with the following format:           *"
echo "*    ------------------------------------------------------------------------  *"
echo "*    alias vps_ip :port  masternode_private_key tx_id          vout            *"
echo "*      mn1 1.2.3.4:12018 5xxxxxxxxxxxxxxxxxxxxx 123456xxxxxxxx 0               *"
echo "* 5. Save the file, exit your wallet and reopen your wallet.                   *"
echo "* 6. Go to menu Tools -> Debug Console                                         *"
echo "* 7. Run the following command: masternode start-alias mn1                     *"
echo "*    Note: replace mn1 with your alias if you used a different alias           *"
echo "********************************************************************************"
echo && echo
echo "********************************************************************************"
echo "* Congratulation! Your setup is completed.                                     *"
echo "*                                                                              *"
echo "* Now you can run commercium-cli                                               *"
echo "* Your datadir is /home/apostille/.commerciumcore                              *"
echo "* Useful command: sudo systemctl status commerciumd                            *"
echo "*                 sudo systemctl restart commerciumd                           *"
echo "*                 sudo systemctl start commerciumd                             *"
echo "********************************************************************************"

