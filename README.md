## About

Fork of transmission 3.0 with skip verify option

## Use carefully!

This modification is only intended to speed up the addition of already verified torrents.  
As an example: One primary instance and several secondary instances attached to the same media.  
After the primary instance verifies the content, you can add the torrent to the secondary instances without additional verification.  
It is recommended to use RO file system mount for secondary instances.

## Thanks to

@yangzhaofeng https://github.com/transmission/transmission/pull/769  
@cfpp2p https://github.com/cfpp2p/transmission - I used this general idea from this fork by adapting it for the new version and refined the performance for large torrents 

### How to skip verify with cli

Use -T option

    transmission-remote -t 1 -T # skip verify for torrent with index 1
### How to skip verify via api (curl)

For api usage is a torrent-skip-verify method
For arguments look at rpc [protocol pt3.1](https://github.com/transmission/transmission/wiki/RPC-Protocol-Specification)

You can use test script below:

    #!/bin/bash
    host="localhost" 
    port="9091" 
    if [ -z "$1" ]; then
            echo "set filename" 
            exit 0
    fi
    token=$(curl http://${host}:${port}/transmission/rpc -s \
            | sed 's/.*Session-Id: \(.*\)<\/code>.*/\1/')
    echo "token is ${token}" 
    read -p "press enter to add torrent" 
    filename=$1
    hash=$(curl http://${host}:${port}/transmission/rpc -s \
            -H "X-Transmission-Session-Id: ${token}" \
            -H "Content-Type: application/json" \
            -d '{"method": "torrent-add", "arguments": {"filename": "'${filename}'"}}'\
            | sed 's/.*hashString\":\"\(.*\)\",\"id\".*/\1/ ') 
    echo "Torrent hashcode is ${hash}" 
    read -p "press enter to verify torrent" 
    curl http://${host}:${port}/transmission/rpc -s \
            -H "X-Transmission-Session-Id: ${token}" \
            -H "Content-Type: application/json" \
            -d '{"method": "torrent-skip-verify", "arguments": {"ids": ["'${hash}'"]}}'
    read -p "press enter to delete torrent" 
    curl http://${host}:${port}/transmission/rpc -s \
            -H "X-Transmission-Session-Id: ${token}" \
            -H "Content-Type: application/json" \
            -d '{"method": "torrent-remove", "arguments": {"ids": ["'${hash}'"]}}'

Visit https://transmissionbt.com/ for more information.

### Building Transmission from Git (first time)

    $ git clone https://github.com/batkom/transmission_skip_verify Transmission
    $ cd Transmission
    $ git submodule update --init
    $ mkdir build
    $ cd build
    $ cmake ..
    $ make
    $ sudo make install

### Building Transmission from Git (updating)

    $ cd Transmission/build
    $ make clean
    $ git pull --rebase --prune
    $ git submodule update
    $ cmake ..
    $ make
    $ sudo make install