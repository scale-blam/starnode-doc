# Starnode command line options

## Main usage

To run a _Node.js_ `app.js` script using _Starnode_

```
$ starnode --user=<USER-ID> [<starnode-options>] app.js [<app-options>]
```

**Warning: any option passed after `app.js` will be ignored by _Starnode_, and passed as is to `app.js`**

## Mandatory options

### user

The _user_ option expects an ID that allows you to be recognized by _Starnode_:

```
--user=<user-id>
```

## Other options

### name

You can indicate a name to differentiate your _Starnode_ instances.

```
--name=<string>
```

### num-threads

By default _Starnode_ uses the number of available CPUs when creating the execution _warp_ threads.

To set a particular number of _warp_ threads use this option: 

```
--num-threads=<integer>
```

### project

_Starnode_ is collecting some information that is sent to ta server in order to dynamically
optimize the application/service execution. To recognize one application from another, _Starnode_
automatically uses the name and version (<name>-<version>) available in the `package.json` found
in the `app.js` directory or its parents. If it is not found, `<unknown>` is used unless the option
below is set.

```
--project=<string>
```

### debug

Enable debugging mode for the current _Starnode_ instance. The debugging mode is intended to be used during
development, additional warnings are reported to help write and debug your code.

```
--debug
```

### expert

The expert mode disables some checks about what can be executed, like for instance the use of requires.
See details in [Limitations](Limitations.md).   
To enable the expert mode, use the option below:

```
--expert
```

### version

Print _Starnode_ version and exit:

```
--version
```

### help

Print help and exit:

```
--help, or --h
```
