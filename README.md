# Tezos Cheat Sheet

## RESOURCES




[Tzscan.io Block explorer](https://tzscan.io/)







[Tezos Betanet Documentation](http://tezos.gitlab.io/betanet/)
[Tezos Alphanet Documentation](http://doc.tzalpha.net/api/rpc.html)






[List of delegation services and their fees](http://www.mytezosbaker.com/bakers-list/)





[Guide to set up a Tezos Client and start baking](https://github.com/tezoscommunity/FAQ/wiki/Build-a-zeronet-node-on-Debian-9/)




[Tezos Ledger Nano S applications](https://github.com/obsidiansystems/ledger-app-tezos/blob/master/README.md) (Wallet and Baking application and a great guide how to start baking) (by ObsidianSys)






[Simple instructions to bake and keep (most of) your tezzies offline](https://github.com/maxtez-raspbaker/tezos-rpi3/wiki/%5B0%5D-Simple-instructions-to-bake-and-keep-(most-of)-your-tezzies-offline)


[Tezos Cycle Baking Stats](https://docs.google.com/spreadsheets/d/1TkU71UPfA8g-zgy1y-wKAA3uOJCZr2LeJpjx05KUCXU/edit#gid=1853124312) (credits to sirneb for keeping this up to date!)

[Further Delegation and Baking Statistics](https://docs.google.com/spreadsheets/d/1Frn5NTHkHIXk0hWXDqqwPTJfxznR3PSQWdJUgA1aqO0/edit#gid=0)


[View detailed baking rights](https://rpc.tezrpc.me/chains/main/blocks/head/helpers/baking_rights?cycle=7) (Change the cycle number!)

[View detailed baking rights](https://rpc.tezrpc.me/chains/main/blocks/head/helpers/endorsing_rights?cycle=7) (Change the cycle number!)


[List of further resources](https://www.tezos.help/)



## CYCLES
The first cycle is defined as cycle 0.
A cycle has 4096 blocks. (0 to 4095)

## SNAPSHOTS
Every cycle 16 snapshots are taken. They occur every 256 blocks. Therefore the first will be at block 255 of the current cycle (The first block is block 0.).
Snapshot 0 (block 255), Snapshot 15 (block 4095).
To calculate the block in the cycle where the snapshot was taken use this formula:

(cycle #) * 4096 + ((snapshot #) + 1) * 256 = (block of the snapshot)
cycle #: Cycle, where the snapshot was taken.

If you want to know what block cycle 8 was taken, you know it was taken in cycle 1. If snapshot 14 is the snapshot chosen by the chain, the block was 1 * 4096 + (14 + 1) * 256.
If you want to know which snapshot was taken:
```
./tezos-client rpc get "/chains/main/blocks/head/context/raw/json/rolls/owner/snapshot/7"
```
If it is not decided yet which snapshot is taken more than one snapshot will be listed.
If we are in cycle 3 and we run this it will return 9.
If we run this for cycle 9 instead of the 7 at the end we will see all the snapshots which have already been taken in cycle 3 (e.g. [0,1,2,...]), but no one has been chosen yet.

Only one of the 16 snapshots per cycle will be chosen randomly.

The snapshot which determines the baking rights for cycle 8 was taken at block 7936.


## BAKING AND ENDORSING RIGHTS:
Every block 80 new Tezos are created if the baker with priority 0 bakes the block. 16 XTZ for the baker and 2 XTZ for each of the 32 endorsers.
The reward is ENDORSEMENT_REWARD = 2 / BLOCK_PRIORITY where block priority starts at 1. (BLOCK_PRIORITY=BAKING_PRIORITY+1)
If the baker with baking priority 9 bakes the block every endorser will only get 0.2 XTZ.
If many blocks are delayed the "health" of the chain gets worse. http://tzscan.io/health

You can see the current baking and endorsing rights in this Excel Sheet. (credits to sirneb!)
https://docs.google.com/spreadsheets/d/1TkU71UPfA8g-zgy1y-wKAA3uOJCZr2LeJpjx05KUCXU/edit#gid=1853124312

The snapshot which determines the baking rights for a cycle is taken 5 cycles before the baking cycle.
In cycle 3 we know the snapshot chosen for cycle 8.
If you want to know which snapshot is chosen for cycle 9, you have to wait for cycle 4.

The following follwos the structure:
```
/chains/main/blocks/head/context/raw/json/rolls/owner/snapshot/<cycle>
```

Get snapshots for cycle 8:
```
./tezos-client rpc get "/chains/main/blocks/head/context/raw/json/rolls/owner/snapshot/8"
```

Get snapshot number 9 for cycle 7:
```
./tezos-client rpc get "/chains/main/blocks/head/context/raw/json/rolls/owner/snapshot/7/9?depth=1"
```

## BAKING RIGHTS
https://rpc.tezrpc.me/chains/main/blocks/head/helpers/baking_rights?cycle=7 (Change the cycle number!)

The baker with priority 0 has the first rights to bake a block.
The number of "preserved cycles" is 5. If a block gets baked in cycle 8 the reward will be spendable in cycle 14.


Get baking rights for cycle 7:
```
./tezos-client rpc get "/chains/main/blocks/head/helpers/baking_rights?cycle=7&max_priority=1" | jq -r '.[] | .delegate' | sort | uniq -c | sort -rnk2
```
https://rpc.tezrpc.me/chains/main/blocks/head/helpers/baking_rights?cycle=7

Get baking rights for cycle 7 for specific address:
```
./tezos-client rpc get /chains/main/blocks/head/helpers/baking_rights?cycle=7 |grep tz3UoffC7FG7zfpmvmjUmUeAaHvzdcUvAj6r -A 1
./tezos-client rpc get '/chains/main/blocks/head/helpers/endorsing_rights?delegate=tz3UoffC7FG7zfpmvmjUmUeAaHvzdcUvAj6r&cycle=7'
```

## ENDORSING RIGHTS

https://rpc.tezrpc.me/chains/main/blocks/head/helpers/endorsing_rights?cycle=7 (Change the cycle number!)

Get endorsing rights for cycle 7:
```
./tezos-client rpc get '/chains/main/blocks/head/helpers/endorsing_rights?cycle=7' | jq '.[] | . as $right | path(.|.slots[]) | $right | .delegate' | sort | uniq -c
```
https://rpc.tezrpc.me/chains/main/blocks/head/helpers/endorsing_rights?cycle=7


Get endorsing rights for cycle 7 for specific address:
```
./tezos-client rpc get /chains/main/blocks/head/helpers/endorsing_rights?cycle=7 |grep tz3UoffC7FG7zfpmvmjUmUeAaHvzdcUvAj6r -A 1
```

## SECURITY DEPOSITS (BONDS)
For baking a block 512 XTZ is needed, for one endorsement it's 64 XTZ.
But there is a ramp-up in the first few cycles.
The endorsement bond increases by 1 XTZ each cycle until it reaches 64.
The baking bond increases by 8 XTZ per cycle until it reaches 512.
So in cycle 7 the endorsement bond is 7 XTZ and the baking bond is 56 XTZ.
The idea behind the ramp-up is to let delegates build up their bonds.

## TEZOS NODE
To set up a node look at this genious guide by fredcy! Thanks!
https://github.com/tezoscommunity/FAQ/wiki/Build-a-zeronet-node-on-Debian-9

If you have an old opam version try this:
```
opam init
opam update
opam switch create 4.06.1
opam switch set 4.06.1
eval $(opam config env)
```

Starting the node:
```
./tezos-node run --rpc-addr localhost:8732
```

Start the node in private mode:
```
./tezos-node run --private-mode --rpc-addr localhost:8732
```

Get current block:
```
./tezos-client rpc get /chains/main/blocks/head/
```
https://rpc.tezrpc.me/chains/main/blocks/head

Get new tz1 address:
```
./tezos-client gen keys <name>
```

Connecting your node to specific other nodes following the structure --private --peer:IP:PORT: 
You can pick the IP addresses from http://tzscan.io/network . If you want to run a private node it is better to connect to your won nodes!
```
./tezos-node run --rpc-addr localhost:8732 --peer 134.119.17.192:9732 --peer 34.245.218.74:9732 --peer 34.244.213.249:9732 --peer 34.245.57.176:9732
```

Listing all connections of your node:
```
./tezos-client rpc get /network/connections
```

This can just be used if you have jq. It lists all incoming connections.
```
.tezos-client rpc get /network/connections | jq '.[] | .incoming, .id_point.addr, .versions[].name'
```
Use this to view your configuration of the node:
```
./tezos-node config
```

Activate your fundraiser account:
```
./tezos-client activate fundraiser account my_account with <activation key from verification site>
```

Import a key (e.g. from ledger nano s): you can choose the alias on your own.
```
./tezos-client import <alias> secret key <secret key>
```

List known Tezos addresses:
```
./tezos-client list known addresses
```
Originate an account
```
./tezos-client originate account secondary_account2 for primary_account2 transferring xxx from primary_account2 --fee 0 --delegate baking_account --delegatable
```

If your node can't establish any connections:
You may consider if you are the right one for a severe long-term Tezos-relationship!
Otherwise:
You might have to open port 9732 (TCP).
Do not open the RPC port, which is 8372. Thsi would not we secure.

There is a bootstrap-option:
```
--no-bootstrap-peers
```

## RUN YOUR NODE IN BACKGROUND MODE (e.g. to be able to close putty and still continue the process)
Use one of these:

1. pm2 (requires npm to install) Has a lot of features e.g. auto restart.
2. tmux
3. screen
4. nohup

## BAKING ON YOUR OWN




How you know if the endorser is running correctly?
The baker-process and accuser-process both display blocks they check.
The endorser-process doesn't display any output.

## THE FOUR TYPES OF OPERATIONS
1. Endorsements
2. ?
3. Nonces & Denouncement
4. Transactions?

## The three types of Tezos addresses

Implicit addresses:

tz1: ed25519

tz2: secp256k1

tz3: p256 (not as secure as ed25519, mainly for mobile devices, they don't need much computing power)

Originated addresses:

kt1: originated accounts (these are smart contracts) (Belong to a implicit address, you need them for staking your coins or smart contracts. An implicit address is the manager of a kt1 address. Every origination burns 0.257 XTZ.)

Implicit accounts can bake their own balance + the balance staked by kt1 addresses and contracts.
Even if a part of the balance is currently bonds tese bonds count to the rolls.
Only implicit addresses can delegate to themselves and bake, but they can not delegate their stake to another address.
Tezos foundation and dls use tz3 addresses.
p256 is still not available on ledger, but it will be available.

## OVER-DELEGATION
The maximum amount a baker can bake per roll is 10000/0.0825*percentage of XTZ baked

The baker needs a minimum of 8.25 percent as security deposit in order to bake in case 100% of all XTZ are taking part in the baking process.
If more Tezos gets baked more security deposits are needed as the single baker can bake and endorse more blocks.
In case 30% of all Tezos is taking part in the baking process 36363 XTZ is the maximum bakeable amount per roll.

## STATISTICS (12th July 2018)
On Tuesday 10th of July the Tezos folder had a size of 1.1GB.
152 mio baking in the cycles 0-6 (Tezos Foundation and DLS).
190 mio will take part in the baking process in cycle 7.
224 mio will take part in the baking process in cycle 8.



## I want to thank the Riot-Tech channel for all of this information.
All the information is theirs! Especially I want to thank:
(not ordered, unfortunately there are many many more and a lot of links and references missing)







sirneb (Baking and Endorsing statistics: https://docs.google.com/spreadsheets/d/1TkU71UPfA8g-zgy1y-wKAA3uOJCZr2LeJpjx05KUCXU/edit#gid=1853124312)

Braveheart

fredcy (Start baking guide: https://github.com/tezoscommunity/FAQ/wiki/Build-a-zeronet-node-on-Debian-9)

stephenandrews (Bakechain (Baking application and marketplace for the average user on windows), TezBox (Wallet) (https://tezbox.com/), rpc.tezrpc.me (public node) and so much more!)

Tezzigator (So much information and more!)

cago (tzscan.io)

maxtez-raspbaker

Martin Eian

Blindripper

wholesum

DefinitelyNotAGoat

bitcom

a_a_a

ricopvt

tomjack

bobbyLiberty