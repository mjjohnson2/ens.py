
# Ethereum Name Service via Python

[![Join the chat at https://gitter.im/ens-py/Lobby](https://badges.gitter.im/ens-py/Lobby.svg)](https://gitter.im/ens-py/Lobby?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

Access the Ethereum Name Service using this python library. Note: **this is a work in progress**

Using this library is not a way to skip learning how ENS works. If you are registering a name, a
small misunderstanding can cause you to lose **all** your deposit.
Go [read about ENS](http://docs.ens.domains/en/latest/userguide.html) first.
Your funds are your responsibility.

## Alpha-quality warning

This is a preview for developers, and an invitation for contributions. Please do not use this in
production until this warning is removed, especially when putting funds at risk. Examples of funds
being at risk include: sending ether/tokens to resolved addresses and participating in name
auctions.

The [nameprep algorithm](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-137.md#name-syntax)
implementation is not well tested, please be cautious and double-check the result through another
channel. If you supply the name in `bytes`, it will be assumed to be UTF-8 encoded, like in
[Ethereum contracts](https://github.com/ethereum/wiki/wiki/Ethereum-Contract-ABI#argument-encoding).
Currently, several of the convenience methods only split on '.' -- instead, UT-46 says that several
other dot characters should be split, too.


## Setup

```
pip install ens
```

Any issues? See [Setup details](#setup-details)

## Usage

All examples in Python 3

### Name info

#### Get address from name

Default to {name}.eth:

```
from ens import ens


# look up the hex representation of the address for a name

eth_address = ens.lookup('jasoncarver.eth')

assert eth_address == '0x5b2063246f2191f18f2675cedb8b28102e957458'


# ens.py will assume you want a .eth name if you don't specify a full name

assert ens.lookup('jasoncarver') == eth_address
```

#### Get name from address

```
domain = ens.reverse('0x5b2063246f2191f18f2675cedb8b28102e957458')


# reverse() also accepts the bytes version of the address

assert ens.reverse(b'[ c$o!\x91\xf1\x8f&u\xce\xdb\x8b(\x10.\x95tX') == domain


# confirm that the name resolves back to the address that you looked up:

assert ens.lookup(domain) == '0x5b2063246f2191f18f2675cedb8b28102e957458'
```

#### Get owner of name

```
eth_address = ens.owner('exchange.eth')
```

### Auctions for names ending in .eth

#### Get auction status

Example with domain 'payment.eth':

```
from ens.registrar import Status


status = ens.registrar.status('payment')


# if you forget to strip out .eth, ens.py will do it for you

assert ens.registrar.status('payment.eth') == status


# these are the possible statuses

assert status in (
  Status.Open,
  Status.Auctioning,
  Status.Owned,
  Status.Forbidden,
  Status.Revealing,
  Status.NotYetAvailable
  )


# if you get the integer status from another source, you can compare it directly

assert Status.Owned == 2
```

#### Start auctions

```
# start one auction (which tips people off that you're interested)

ens.registrar.start('you_saw_him_repressin_me_didnt_ya')


# start many auctions (which provides a bit of cover)

ens.registrar.start(['exchange', 'tickets', 'payment', 'trading', 'registry'])
```

#### Bid on auction

Bid on a 'trading.eth' with 5211 ETH, and secret "I promise I will not forget my secret":

```
from web3utils import web3

ens.registrar.bid(
      'trading',
      web3.toWei('5211', 'ether'),
      "I promise I will not forget my secret",
      transact={'from': web3.eth.accounts[0]}
      )
```
(if you want to "mask" your bid, set a higher value in the transact dict)

#### Reveal your bid

You must **always** reveal your bid, whether you won or lost.
Otherwise you will lose the full deposit.

Example of revealing your bid on 'registry.eth' with 0.01 ETH, and secret
"For real, though: losing your secret means losing ether":

```
ens.registrar.reveal(
      'registry',
      web3.toWei('0.01', 'ether'),
      "For real, though: losing your secret means losing ether",
      transact={'from': web3.eth.accounts[0]}
      )
```

#### Claim the name you won

aka "Finalize" auction, which makes you the owner in ENS.

```
ens.registrar.finalize('gambling')
```

#### Get detailed information on an auction

aka "entries" for an auction

```
entries = ens.registrar.entries('ethfinex')


# confirm the auction is closed

assert entries[0] == Status.Owned


# find out the owner of the auction Deed --
see [docs on the difference](http://docs.ens.domains/en/latest/userguide.html#managing-ownership)
between owning the name and the deed

assert entries[1].owner() == '0x9a02ed4ca9ad55b75ff9a05debb36d5eb382e184'


# when was the auction completed? (a timezone-aware datetime object)

assert str(entries[2]) == '2017-06-05 08:10:03+00:00'


# how much is held on deposit?

from decimal import Decimal

assert web3.fromWei(entries[3], 'ether') == Decimal('0.01')


# what was the highest bid?

assert web3.fromWei(entries[4], 'ether') == Decimal('201709.02')
```

## Setup details

### If Python 2 is your default, or you're not sure

In your shell
```
if pip --version | grep "python 2"; then
  python3 -m venv ~/.py3venv
  source ~/.py3venv/bin/activate
fi
```

### Now, with Python 3

In your shell: `pip install ens`

*ens.py* requires an up-to-date Ethereum blockchain, preferably local. If your setup isn't working,
try running `geth --fast` until it's fully-synced. I highly recommend using the default IPC
communication method, for speed and security.

### "No matching distribution found for ens"

If you are seeing something like:
```
Collecting ens
  Could not find a version that satisfies the requirement ens (from versions: )
No matching distribution found for ens
```

Then retry the first Setup section, to make sure you're in Python 3

### Optionally, a custom web3 provider

In Python:

```
from ens import ENS
from web3utils import web3
from web3 import IPCProvider 

web3.setProvider(IPCProvider('/your/custom/ipc/path'))

ens = ENS(web3)
```



## Developer Setup

```
git clone git@github.com:carver/ens.py.git
cd ens.py/

python3 -m venv venv
. venv/bin/activate

pip install -e .
pip install -r requirements-dev.txt
```

### Why does ens.py require python 3?

Because [web3utils requires python 3](https://github.com/carver/web3utils.py#why-is-python-3-required).
Plus, Ethereum is brand new. You shouldn't have any legacy
code that requires you to use a ~10-year-old python version.
