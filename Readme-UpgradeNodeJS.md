# Upgrade Node.js on the BeagleBone Black/Green from 0.12 to 4.x/6.x

## Why?

### 1) Performance in theory more recent versions are more performant.

I measured the before (node 0.12.x) and after (node 4.6.1)

| Measure                        | v0.12.x | v4.6.1 | 
| --- | ---: | ---: |
| Node Process CPU usage  | 32.5 % | 29.4 % |
| Node Process Memory usage | 17.6 % | 7.9 % |
| `forever list` complete time     | 10.78 sec| 17.6 sec |

The node process I used was a process that does a mix of things, lots of mathematical calculation and some file IO.  Note that `forever list` was using a different version on the second run, which may explain why it did so poorly.

### 2) Node v0.12 is old

Node v0.12:
* will [no longer be supported](https://github.com/nodejs/LTS) from 2017 onwards.
* is largely based on ES5


## Upgrading

These instructions are based on [these](http://nodered.org/docs/hardware/beagleboneblack).

  curl -sL https://deb.nodesource.com/setup_4.x | sudo -E bash -
  apt-get install -y build-essential nodejs

