# GeekCash Sentinel Node.js


This guide covers installing Sentinel onto an existing Masternode in Ubuntu 14.04 / 16.04.

## Installation

### 1. Install Prerequisites

Make sure Node v8.x.x or above is installed:
```js
curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash -
sudo apt-get install -y nodejs
```
```js
sudo apt-get install -y build-essential libzmq3-dev

```
Install PM2:
```js
# 1. Add the PM2 repository signing key
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv D1EA2D4C

# 2. Add the PM2 repository
echo "deb http://apt.pm2.io/ubuntu stable main" | sudo tee /etc/apt/sources.list.d/pm2.list

# 3. Update list of available packages
sudo apt-get update

# 4. Install PM2
sudo apt-get install pm2
```

Make sure the local GeekCash daemon running is at least version 1.0.1.2 (1000102)
```
geekcash-cli getinfo
```

### 2. Install Sentinel

Clone the Sentinel repo and install Node packages.
```
git clone https://github.com/geekcash/node-sentinel.git && cd node-sentinel

```

Install node modules
```
npm i
```

If you get an error like this, install this command:

```
sudo apt-get install libboost-all-dev -y
```

```
> zmq@2.15.3 install /root/node-sentinel/node_modules/zmq
> node-gyp rebuild

Traceback (most recent call last):
  File "/usr/lib/node_modules/npm/node_modules/node-gyp/gyp/gyp_main.py", line 13, in <module>
    import gyp
  File "/usr/lib/node_modules/npm/node_modules/node-gyp/gyp/pylib/gyp/__init__.py", line 8, in <module>
    import gyp.input
  File "/usr/lib/node_modules/npm/node_modules/node-gyp/gyp/pylib/gyp/input.py", line 5, in <module>
    from compiler.ast import Const
ImportError: No module named compiler.ast
gyp ERR! configure error 
gyp ERR! stack Error: `gyp` failed with exit code: 1
gyp ERR! stack     at ChildProcess.onCpExit (/usr/lib/node_modules/npm/node_modules/node-gyp/lib/configure.js:345:16)
gyp ERR! stack     at emitTwo (events.js:126:13)
gyp ERR! stack     at ChildProcess.emit (events.js:214:7)
gyp ERR! stack     at Process.ChildProcess._handle.onexit (internal/child_process.js:198:12)
gyp ERR! System Linux 4.4.0-127-generic
gyp ERR! command "/usr/bin/node" "/usr/lib/node_modules/npm/node_modules/node-gyp/bin/node-gyp.js" "rebuild"
gyp ERR! cwd /root/node-sentinel/node_modules/zmq
gyp ERR! node -v v8.12.0
gyp ERR! node-gyp -v v3.8.0
gyp ERR! not ok 
npm WARN node-sentinel@1.0.1 No description
npm WARN node-sentinel@1.0.1 No repository field.

npm ERR! code ELIFECYCLE
npm ERR! errno 1
npm ERR! zmq@2.15.3 install: `node-gyp rebuild`
npm ERR! Exit status 1
npm ERR! 
npm ERR! Failed at the zmq@2.15.3 install script.
npm ERR! This is probably not a problem with npm. There is likely additional logging output above.

npm ERR! A complete log of this run can be found in:
npm ERR!     /root/.npm/_logs/2018-09-15T07_22_16_232Z-debug.log
```




### 3. Create config.js

```
cp -i config_example.js config.js
```

Get RPC User & Password in `.geekcash/geekcash.conf`

```
cd ../
cat .geekcash/geekcash.conf
```
```
rpcuser=root
rpcpassword=pass
rpcport=6888
listen=1
server=1
daemon=1
logtimestamps=1
maxconnections=64
txindex=1
masternode=1
externalip=x.x.x.x:6889
masternodeprivkey=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

Edit config.js
```
cd node-sentinel
nano config.js
```

```js

version:'1.2.1',
coins: [
        {
            id: 'MN-01',
            rpc: {
                url: 'http://127.0.0.1:6888/',
                method: 'POST',
                auth: {
                    username: 'root',
                    password: 'pass'
                },
            }
        },
        {
            id: 'MN-02',
            rpc: {
                url: 'http://127.0.0.1:6888/',
                method: 'POST',
                auth: {
                    username: 'root',
                    password: 'pass'
                },
            }
        },
        // {
        //     id: 'MN-03',
        //     rpc: {
        //         url: 'http://127.0.0.1:6888/',
        //         method: 'POST',
        //         auth: {
        //             username: 'root',
        //             password: 'pass'
        //         },
        //     }
        // },

    ]

```

### 4. Allow RPC IP
If you run sentinel for multiple masternodes, you need to configure rpcallowip to allow sentinel to connect to this MN and restart GeekCash Daemon.
```
rpcuser=root
rpcpassword=pass
rpcallowip=[IP of sentinel]
rpcport=6888
listen=1
server=1
daemon=1
logtimestamps=1
maxconnections=64
txindex=1
masternode=1
externalip=8.8.8.8:6889
masternodeprivkey=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

Ex: your sentinel run at IP: 9.9.9.9, others MN config

```
rpcuser=root
rpcpassword=pass
rpcallowip=9.9.9.9
rpcport=6888
listen=1
server=1
daemon=1
logtimestamps=1
maxconnections=64
txindex=1
masternode=1
externalip=8.8.8.8:6889
masternodeprivkey=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```
```
geekcash-cli stop
geekcashd
```

### 5. Run app
```
pm2 start ./app.js --name sentinel
```

Setup startup script
```js
// Restarting PM2 with the processes you manage on server boot/reboot is critical. To solve this, just run this command to generate an active startup script:
pm2 startup
```

### MIT License

Copyright (c) 2018 GeekCash Team (https://geekcash.org)

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.