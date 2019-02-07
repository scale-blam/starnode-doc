# Starnode command line options

## Main usage

To run a nodejs _app.js_ script using Starnode

```
$ starnode [<starnode-options>] app.js [<app-options>]
```

A typical Starnode command is

```
$ starnode --user=<user-id> app.js <app-options>
```

**Warning: any option passed after _app.js_ will be ignored by Starnode, and passed as is to _app.js_**

## Mandatory options

### user

The _user_ option expects an ID that allows you to be recognized by Starbase:

```
--user=<user-id>
```

### project (if no package.json)

Starnode is collecting some information that is sent to the Starbase server in order to dynamically
optimize the application/service execution. To recognize one application from another, Starnode
automatically uses the name and version (name-version) available in the package.json when launching the
command. If no package.json is found, please use the option below to set an indicative name that
represents your application/service.

```
--project=<string>
```

## Other options

### name

You can indicate a name to differentiate your Starnode instances.

```
--name=<string>
```

### numFlows

By default Starnode uses the number of available CPUs when creating the execution flows.

To set a particular number of flows use this option: 

```
--numFlows=<integer>
```

### debug

Enable debugging mode for the current Starnode instance. The debugging mode is intended to be used during
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

Print Starnode version and exit:

```
--version
```

### help

Print help and exit:

```
--help, or --h
```
