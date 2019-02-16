# Getting Started

## Requirements

Starnode is heavily tested on linux machines, but it must works on _macOS_ as well.
Please, let's us know if you encounter issues on this OS.

You need to have:
- [_Node.js_](https://nodejs.org/en/download/) 8 or newer ;
- An up-to-date _npm_ (it is installed with _Node.js_ and should be
  updated with: `npm install -g npm`).

## Starnode installation

To install Starnode

```
$ npm install -g https://starbase.scaledynamics.io/download/public/starnode-latest.tgz
```

A `sudo` could be required to install it:

```
$ sudo npm install -g https://starbase.scaledynamics.io/download/public/starnode-latest.tgz
```

## Sample codes

Download and install:

```
$ git clone git://github.com/ScaleDynamics/starnode-samples.git
$ cd starnode-samples
$ npm install
```

### Compute prime factorizations with _Starnode_

- Run with _Node_
```
$ node prime/prime-node.js
```

- Run with _Starnode_ (replace `<user-id>` with your real user ID, create or log into
[your account](https://www.scaledynamics.io) to get it):
```
$ starnode --user <user-id> prime/prime-starnode.js 
```

Compare execution time display at the end of each execution. _E.g._ below is what we get on
a 6 CPUs computer:

```
$ node prime/prime-node.js 
Computing 256 prime factorizations...
[...]
Prime factors of 9007199254041245 are 5,862117,2089553797
Prime factors of 9007199254041246 are 2,3,3,3,12241,13626336589
256 prime factorizations computed in 12.235 seconds by Node with its unique thread on a 6 CPU(s) host

$ starnode --user=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx prime/prime-starnode.js 
Computing 256 prime factorizations...
[...]
Prime factors of 9007199254041243 are 3,3002399751347081
Prime factors of 9007199254041227 are 9007199254041227
256 prime factorizations computed in 2.525 seconds by Starnode with 6 warp thread(s) on a 6 CPU(s) host
```

To see how easy it is to modify _JavaScript_ with the
[_Warp_ API](https://www.npmjs.com/warp), you can compare code
between the _Node_ and the _Starnode_ version with:

```
$ diff --side-by-side --left-column prime/prime-node.js prime/prime-starnode.js
```

## Next steps

Great! You can now read the other documentations to try Starnode on your own scripts:

- [To discover the _Warp API_](https://npmjs.com/warp),
- [To get all options of starnode command line](doc/Cli.md),
- [To understand Starnode current limitations](doc/Limitations.md).
