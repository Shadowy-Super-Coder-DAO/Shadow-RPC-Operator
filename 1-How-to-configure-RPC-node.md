# RPC Node config for Equinix Metal

IMPORTANT: This guide is specifically for Equinix Machines from the Solana Reserve pool accessed through he Solana Foundation Server Program. https://solana.foundation/server-program

You must be running Ubuntu 20.04

So you have your shiny new beast of a server. Let's make it a Shadow Operator RPC.

First things first - OS security updates
```
sudo apt update
sudo apt upgrade (Y)

Configuration file '/etc/cloud/cloud.cfg'
 ==> Modified (by you or by a script) since installation.
 ==> Package distributor has shipped an updated version.
   What would you like to do about it ?  Your options are:
    Y or I  : install the package maintainer's version
    N or O  : keep your currently-installed version
      D     : show the differences between the versions
      Z     : start a shell to examine the situation
 The default action is to keep your current version.
*** cloud.cfg (Y/I/N/O/D/Z) [default=N] ? 


sudo apt dist-upgrade
```
create user sol

```
adduser sol

usermod -aG sudo sol

su - sol

```
Partition hard drive for RPC
Partition NVME into 420gb (swap) and 3000gb (ledger and accounts)

adding new process using GPT partition with gdisk for larger filessytems. Make larger 3.5 (or 3.8) TB drive via gdisk then partition using fdisk as normal. You have to delete the original GPT in order to select partition 1 with fdisk

Enter the "n" then hit enter
Etner the "1" then hit enter...and so on
```
sudo gdisk /dev/nvme0n1
n, 1, p, 2048, [max secor available], 8300, p, w
```
p doesn't seem to be a viable option? n, 1, 2048, enter, enter, p, w, Y

note the first step in the next section is deleting the partition we just created above
```
sudo fdisk /dev/nvme0n1
d, n, p, 1 or 2, default sector, +3000GB, n, p, 1 or 2, default sector, +420GB, w
```
p doesn't seem to be a viable option here either. d, n, 1, enter, +3000G, n, 2, enter, +420G, w

Now make filessytems, directories, delete and make new swap, etc.
```
sudo fdisk -l 

sudo mkfs -t ext4 /dev/nvme0n1p1

sudo mkfs -t ext4 /dev/nvme0n1p2

sudo mkdir /mnt/ (File exists?)

sudo mkdir /mnt/ramdrive

sudo mkdir /mt/

sudo mkdir /mt/ledger

sudo mkdir /mt/ledger/validator-ledger

sudo mkdir /mt/solana-accounts

sudo mkdir ~/log
```
Discover the swap directory, turn it off, make a new one and turn it on
```
sudo swapon --show

```
You need to look at the directory and pick the correct /dev/sd*

It could be /dev/sdb2 or /dev/sdc2 so edit the next line below to the proper sd**
```
sudo swapoff /dev/sda2

sudo sed --in-place '/swap.img/d' /etc/fstab

sudo mount /dev/nvme0n1p2 /mnt/

sudo mount /dev/nvme0n1p1 /mt

sudo dd if=/dev/zero of=/mnt/swapfile bs=1M count=350k
```
it can take up to 5 minutes for the machine to make this size swapfile. sit tight.

next is setting permissions and adding the swapfile to fstab, then edit the swapiness to 30.
```
sudo chmod 600 /mnt/swapfile

sudo mkswap /mnt/swapfile

echo 'vm.swappiness=30' | sudo tee --append /etc/sysctl.conf > /dev/null

sudo sysctl -p

sudo swapon --all --verbose
```
capture nvme0n1p1 and nvme0n1p2 UUIDs to edit into /etc/fstab
```
lsblk -f
```
copy the section that looks like this and past it into a notepad (or VScode, etc) so that you can copy/past into fstab properly. We just need the UUID's so in the example below copy "5c24e241-239c-4aa5-baa6-fbb6fb44a847" and "87645b08-85c2-4fe2-9974-1bda4de317d9" and note which partition each belongs to (/mt and /mnt respectively)
```
nvme0n1
├─nvme0n1p1 ext4         5c24e241-239c-4aa5-baa6-fbb6fb44a847    2.8T     0% /mt
└─nvme0n1p2 ext4         87645b08-85c2-4fe2-9974-1bda4de317d9    9.5G    88% /mnt
```
These UUID above need to be edited into the fstab config below
```
sudo nano /etc/fstab
```
dump this into fstab below the current UUIDs. delete or hash out the old dwap UUID if needed. Leave the first UUIDs (OS related), just append these lines under whatever current UUIDs are listed as the ones already in the file are boot/OS related.
also make sure UUID is correct as they can change

Copy the below into a notepad along with the above "lsblk -f" information and then update the UUIDs to be the new ones from the current RPC. THen you paste this into the fstab file mentioned above
```
#GenesysGo RPC config
UUID=5c24e241-239c-4aa5-baa6-fbb6fb44a847 /mt  auto nosuid,nodev,nofail 0 0
UUID=87645b08-85c2-4fe2-9974-1bda4de317d9 /mnt  auto nosuid,nodev,nofail 0 0
#ramdrive and swap
#tmpfs /mnt/ramdrive tmpfs rw,size=50G 0 0
/mnt/swapfile none swap sw 0 0
```
save / exit
ctrl+s, ctrl+x

But Wait - what was that ramdrive and tmpfs stuff? Leave it for now. That is an performance enhancement option that will be covered in later documentation. In short, it's for running the solana-accounts inside the memory of the server versus on the hard drive. More on this later.

now edit permissions and make sure user sol is the owner for solana directories
```
sudo chown sol:sol /mt/solana-accounts - this directory doesn't exist, even though we already made it above?

sudo chown sol:sol /mt/ledger - same?

sudo chown sol:sol ~/log

sudo chown sol:sol /mt/ledger/validator-ledger - this one too?
```
mount everything
```
sudo mount --all --verbose
```

Set up the firewall / ssh
```
sudo snap install ufw

sudo ufw enable

sudo ufw allow ssh
```
dump this entire command block
```
sudo ufw allow 80;sudo ufw allow 80/udp;sudo ufw allow 80/tcp;sudo ufw allow 53;sudo ufw allow 53/tcp;sudo ufw allow 53/udp;sudo ufw allow 8899;sudo ufw allow 8899/tcp;sudo ufw allow 8900/tcp;sudo ufw allow 8900/udp;sudo ufw allow 8901/tcp;sudo ufw allow 8901/udp;sudo ufw allow 9900/udp;sudo ufw allow 9900/tcp;sudo ufw allow 9900;sudo ufw allow 8899/udp;sudo ufw allow 8900;sudo ufw allow 8000:8020/tcp;sudo ufw allow 8000:8020/udp
```
# Install Solana CLI! Don't forget to check for current version (1.8.10 as of 12/14/21)

these are three separate commands below:

```
sh -c "$(curl -sSfL https://release.solana.com/v1.8.10/install)"

export PATH="/home/sol/.local/share/solana/install/active_release/bin:$PATH"

solana-gossip spy --entrypoint entrypoint.mainnet-beta.solana.com:8001
```
if the machine is gossiping without any errors it can be spun up on the mainnet to start reading the chain.

exit gossip with ctrl + c

now create keys.

RPCs use throw away keys. These keys allow and RPC to be fully functional but do not need funds and do not need to be saved (because you can just make new ones if you need to ). You do not need to set a password for the keys. No need to copy seed phrases. You do not need a wallet-keypair if just RPC. **Do not move SOL into these wallets. This is not a validator**
```
solana-keygen new -o ~/validator-keypair.json

solana config set --keypair ~/validator-keypair.json

solana-keygen new -o ~/vote-account-keypair.json
```
making system services (sol.service and systuner.service) and the startup script.

this is the solana-validator start up shell script which the system service (sol.service) will reference
```
sudo nano ~/start-validator.sh
```
dump this into start-validator.sh:

```
#!/bin/bash
# v0.5 Shadow Node ( updated 12/14/2021)
export SOLANA_METRICS_CONFIG=host=https://metrics.solana.com:8086,db=mainnet-beta,u=mainnet-beta_write,p=password
PATH=/home/sol/.local/share/solana/install/active_release/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin
export RUST_BACKTRACE=1
export RUST_LOG=solana=info,solana_core::rpc=debug
exec solana-validator \
    --identity ~/validator-keypair.json \
    --entrypoint entrypoint.mainnet-beta.solana.com:8001 \
    --entrypoint entrypoint2.mainnet-beta.solana.com:8001 \
    --entrypoint entrypoint3.mainnet-beta.solana.com:8001 \
    --entrypoint entrypoint4.mainnet-beta.solana.com:8001 \
    --entrypoint entrypoint5.mainnet-beta.solana.com:8001 \
    --known-validator 7cVfgArCheMR6Cs4t6vz5rfnqd56vZq4ndaBrY5xkxXy \
    --known-validator DDnAqxJVFo2GVTujibHt5cjevHMSE9bo8HJaydHoshdp \
    --known-validator Ninja1spj6n9t5hVYgF3PdnYz2PLnkt7rvaw3firmjs \
    --known-validator wWf94sVnaXHzBYrePsRUyesq6ofndocfBH6EmzdgKMS \
    --known-validator 7Np41oeYqPefeNQEHSv1UDhYrehxin3NStELsSKCT4K2 \
    --known-validator GdnSyH3YtwcxFvQrVVJMm1JhTS4QVX7MFsX56uJLUfiZ \
    --known-validator DE1bawNcRJB9rVm3buyMVfr8mBEoyyu73NBovf2oXJsJ \
    --known-validator CakcnaRDHka2gXyfbEd2d3xsvkJkqsLw2akB3zsN1D2S \
    --rpc-port 8899 \
    --dynamic-port-range 8002-8020 \
    --no-port-check \
    --gossip-port 8001 \
    --no-untrusted-rpc \
    --no-voting \
    --private-rpc \
    --rpc-bind-address 0.0.0.0 \
    --rpc-send-retry-ms 100 \
    --enable-cpi-and-log-storage \
    --enable-rpc-transaction-history \
    --enable-rpc-bigtable-ledger-storage \
    --rpc-bigtable-timeout 300 \
    --account-index program-id \
    --account-index spl-token-owner \
    --account-index spl-token-mint \
    --rpc-pubsub-enable-vote-subscription \
    --no-duplicate-instance-check \
    --wal-recovery-mode skip_any_corrupted_record \
    --vote-account ~/vote-account-keypair.json \
    --log ~/log/solana-validator.log \
    --accounts /mt/solana-accounts \
    --ledger /mt/ledger/validator-ledger \
    --limit-ledger-size 700000000 \
    --rpc-pubsub-max-connections 1000 \

```
save / exit (ctrl+o, ctrl+x) (you changed it to nano from vim ;)

make executable
```
sudo chmod +x ~/start-validator.sh
```
change the ownership to user sol
```
sudo chown sol:sol ~/start-validator.sh
```
create system service - sol.service (run on boot, auto-restart when sys fail) 
```
sudo nano /etc/systemd/system/sol.service
```
dump this into file:
```
[Unit]
Description=Solana Validator
After=network.target
Wants=systuner.service
StartLimitIntervalSec=0

[Service]
Type=simple
Restart=on-failure
RestartSec=1
LimitNOFILE=1000000
LogRateLimitIntervalSec=0
User=sol
Environment=PATH=/bin:/usr/bin:/home/sol/.local/share/solana/install/active_release/bin
Environment=SOLANA_METRICS_CONFIG=host=https://metrics.solana.com:8086,db=mainnet-beta,u=mainnet-beta_write,p=password
ExecStart=/home/sol/start-validator.sh

[Install]
WantedBy=multi-user.target
```
save/exit (ctrl+o, ctrl+x)

make system tuner service - systuner.service
```
sudo nano /etc/systemd/system/systuner.service
```
dump this into file:
```
[Unit]
Description=Solana System Tuner
After=network.target
[Service]
Type=simple
Restart=on-failure
RestartSec=1
LogRateLimitIntervalSec=0
ExecStart=/home/sol/.local/share/solana/install/active_release/bin/solana-sys-tuner --user sol
[Install]
WantedBy=multi-user.target
```
save/exit (ctrl+o, ctrl+x)

reload the system services
```
sudo systemctl daemon-reload
```
log rotation for ~/log/solana-validator.log
```
sudo nano /etc/logrotate.d/solana
```
dump this into file:
```
/home/sol/log/solana-validator.log {
  su sol sol
  daily
  rotate 1
  missingok
  postrotate
    systemctl kill -s USR1 sol.service
  endscript
}
```
save/exit (ctrl+o, ctrl+x)

reset log rotate
```
sudo systemctl restart logrotate
```

CPU to performance mode (careful with this)
```
sudo apt-get install cpufrequtils

echo 'GOVERNOR="performance"' | sudo tee /etc/default/cpufrequtils

sudo systemctl disable ondemand
```
modifications to sysctl.conf
```
sudo nano /etc/sysctl.conf
```
edit into bottom of file

```
# other tunings with influence and thanks to the awesome team at rpcpool (aka Triton)
# sysctl_optimisations:
vm.max_map_count=1000000
vm.swappiness=20
kernel.hung_task_timeout_secs=300
vm.stat_interval=10
vm.dirty_ratio=40
vm.dirty_background_ratio=10
vm.dirty_expire_centisecs=36000
vm.dirty_writeback_centisecs=3000
vm.dirtytime_expire_seconds=43200
kernel.timer_migration=0
# A suggested value for pid_max is 1024 * <# of cpu cores/threads in system>
kernel.pid_max=49152
net.ipv4.tcp_fastopen=3
# solana systuner
net.core.rmem_max=134217728
net.core.rmem_default=134217728
net.core.wmem_max=134217728
net.core.wmem_default=134217728
```
save/exit (ctrl+o, ctrl+x)

# start up and test

```
sudo systemctl enable --now systuner.service

sudo systemctl status systuner.service

sudo systemctl enable --now sol.service

sudo systemctl status sol.service
```
or this (prefer the above - the option to use bash is just for debugging)
```
./start-validator.sh
```
tail log to make sure it's fetching snapshot and working
```
sudo tail -f ~/log/solana-validator.log
```
The result should be the machine start tailing the validator log. It can take up to 20 minutes to download a snapshot and begin catching up. The catchup can take up to 45 minutes as well. You can run healthchecks to know when the machine is on the top of the chain (healthy and ready to serve data) by using some of the below commands:

Healthcheck - you want this to return the work "Ok"

If can also return a 'behind by x number of slots" which means it behind the "tip" of the chain by that many slots. Nodes can sometimes fall a little behind and that's normal. Anything above about 100 behind mean you will risk serving stale data.

It can take half an hour before this healthcheck reports slots. Prior it may just say "connection refused." That's normal, give the RPC time to download the data, index the data, and catch up to the top of the chain.
```
curl http://localhost:8899 -k -X POST -H "Content-Type: application/json" -d '
  {"jsonrpc":"2.0","id":1, "method":"getHealth"}
'
```

Tracking root slot
```
timeout 120 solana catchup --our-localhost=8899 --log --follow --commitment root
```
curl for getBlockProduction - this is a simple curl and calls for a little bit larger JSON data response. It should be nearly instant. If it isn't there is a problem.
```
curl http://localhost:8899 -k -X POST -H "Content-Type: application/json" -H "Referer: SSCLabs" -d '{"jsonrpc":"2.0","id":1, "method":"getBlockProduction"}
'
```

Further health checks coming soon including health checks for archival data. Shadow Nodes will store all transactions back to the genesis block 0. More curls will be placed here to make sure your node properly accesses archival. 


