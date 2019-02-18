# Lisinski validator guide

## Generating 0x385 address

Generate an address with the `385` prefix. You can use the VanityEth tool from https://github.com/MyEtherWallet/VanityEth. 

Usage:

```bash
$ vanityeth -i 385
```

Store your private key somewhare safe, and mark the public address for later use.

Example result (do not use this address!):

```bash
{"address":"0x38503ad719619db92b2b735127c709331fcb286d","privKey":"e8a90f4fb3d00205bffae7a57fd92c44781350bb39dc8178eb607b3aa1e38b8c"}
```

Store the `"privKey"` part into a new file named `pk.txt` and don't misplace it :)

## Installing go-ethereum client

Choose your favorite installation procedure from https://github.com/ethereum/go-ethereum/wiki/Installing-Geth and proceed with the necessary steps.

After the installation if you run `geth version` you should see something like:

```bash
$ geth version
Geth
Version: 1.8.22-stable
Git Commit: 7fa3509e2eaf1a4ebc12344590e5699406690f15
Architecture: amd64
Protocol Versions: [63 62]
Network Id: 1
Go Version: go1.10.4
Operating System: linux
GOPATH=
GOROOT=/usr/lib/go-1.10
```

## Initializing Lisinski genesis file and 0x385 account

Download the Lisinski genesis file and place it in your `$HOME` directory (or where you see fit):

```bash
$ cd $HOME && wget https://raw.githubusercontent.com/hpoa/lisinski/master/geth/lisinski.genesis
```

Initialize the Ethereum blockchain storage with the downloaded genesis file:

```bash
$ geth init lisinski.genesis
```

We must now import your `0x385` sealer account private key file generated from the first step (`pk.txt` file) and choose a safe unlock password when asked:

```bash
$ geth account import pk.txt
```

After that you should delete the `pk.txt` file from your server (but before that, store it safely somewhere else - e.g. https://coinsutra.com/store-private-keys-seed-key/)

Store your account unlocking password into a new file (we name it `p` in this guide), or even better, you can provide the password manually when you start the `geth` service (but then you will be required to provide it each restart).

## Starting the geth node

Finally, to start the `geth` use following commmand in which you replace the `<your-sealing-address>` with the public address of the private key from `pk.txt`- again from the first step, and you can also replace `<your-node-name>` with arbitrary node name (for identifying it in  https://status.lisinski.online):

```bash
$ geth --networkid 385 --bootnodes "enode://429862a4e77fc45336173af866eeca74cc2ff686c009cb3ee2cc176b7764759ec9ab906af5a979283886af019527b8eb70f2ff094dab8155ae5cf104460acb6d@159.89.8.217:23453,enode://14dc7a805498052fd7f6781ee7e4e43ce67398a694048225406b5180ac65d266a972ea142c720aeb4191e077cb1474d40af855495a11a494cadeeef276cd2a4c@142.93.230.46:30303" --unlock "<your-sealing-address>" --password p --mine --ethstats "<your-node-name>:Ignatius1819@status.lisinski.online" --port 23453
```

You can choose a random valid port number or omit the `--port` argument for the default `30303` port. If you wish to provide the password manually so you can ommit the `--password p` argument. But beware, if anything goes wrong and your node restarts it will not be able to seal blocks until you manually unlock it. So it is better to provide a password file and modify its permission so only your user can access it with:

```bash
$ chmod 600 p
```



## Creating a `systemd` service

If everything went OK with starting of the node, you will now create a `systemd` service so it can run in the background, start on server boot, and restart itself automatically if it crashes. 

Since it will automatically in the background we must provide a password file and argument (`--password <filename>` ) or manually unlock the account in the `geth` console with `web3.personal.unlockAccount('0x385...', 'mypassword')` on each `geth` launch (this is not recommended since the sealing will stall on each `geth` process restart).

To create a system service we must provide a service descriptor file:

```
[Unit]
Description=go-ethereum node
After=network.target

[Service]
Type=simple
ExecStart=/usr/bin/geth --datadir /home/<username>/.ethereum --networkid 385 --bootnodes "enode://429862a4e77fc45336173af866eeca74cc2ff686c009cb3ee2cc176b7764759ec9ab906af5a979283886af019527b8eb70f2ff094dab8155ae5cf104460acb6d@159.89.8.217:23453,enode://14dc7a805498052fd7f6781ee7e4e43ce67398a694048225406b5180ac65d266a972ea142c720aeb4191e077cb1474d40af855495a11a494cadeeef276cd2a4c@142.93.230.46:30303" --unlock "<your-sealing-address>" --password /home/<username>/p --mine --ethstats "<your-node-name>:Ignatius1819@status.lisinski.online" --port 23453
User=<username>
Group=<username>
Restart=always
RestartSec=10

[Install]
WantedBy=default.target
```

Replace the `<your-node-name>` and `<your-sealing-address>` as before, and `<username>` with your username in all 4 places.

Place the file in `/lib/systemd/system/geth.service` and apply the changes with:

```bash
$ sudo systemctl daemon-reload
$ sudo systemctl enable geth.service
```

Finally, you can start the service with:

```bash
$ sudo systemctl start geth
```


