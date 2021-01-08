# Setting up an XMR miner

The reason that I do this is that if I'm running a VPS I want to be 100% of my money. This helps me recoup about 50¢ off my $5/mo bill.

This would not be immediately profitable if you are renting server space exclusively to do this.

I set the miner priority to lowest possible settings so any other process takes priority, and I limit it to 75% of the total CPU power. This way I don't piss off the VPS host maxing it out for months on end.

First get docker installed on your server or machine of choice.

## First Set up a monero wallet

The easiest way is via the cli.

1.  create the wallet
2.  save the restore seed in a safe place

```bash
sudo apt-get install git build-essential cmake libuv1-dev libssl-dev libhwloc-dev

git clone https://github.com/xmrig/xmrig.git

cd xmrig && mkdir build && cd build

cmake ..

make
```

Now let's get set up with a mining pool we are going to use 

<https://supportxmr.com/>

## Configuring xmrig

Go here to create config file

<https://xmrig.com/wizard>

1.  new configuration
2.  add pool - supportXMR.com
3.  enter your wallet address
4.  optionally add a worker name that will run in processes. (I'll call mine xmr)
5.  click backends - Select the appropriate backends. (just CPU for VPSs)
6.  click misc - donate level 3-5%
7.  be sure to change password to the worker name

Now save the file as `config.json` to the build folder. 

### Setting a CPU Limit

First download `cpulimit`

```bash
sudo apt-get install cpulimit
```

Now run xmrig through cpulimit

```bash
screen -S xmrig

cpulimit -l 75 -b ./xmrig
```

-   the `xxxxx` is your pid number
-   `-l` is 75% of cpu
-   `-b` saves to run it in the background
