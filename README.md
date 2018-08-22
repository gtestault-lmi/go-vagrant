![Release](https://img.shields.io/github/release/bmatcuk/go-vagrant.svg?branch=master)
[![Build Status](https://travis-ci.com/bmatcuk/go-vagrant.svg?branch=master)](https://travis-ci.com/bmatcuk/go-vagrant)
[![codecov.io](https://img.shields.io/codecov/c/github/bmatcuk/go-vagrant.svg?branch=master)](https://codecov.io/github/bmatcuk/go-vagrant?branch=master)
[![GoDoc](https://godoc.org/github.com/bmatcuk/go-vagrant?status.svg)](https://godoc.org/github.com/bmatcuk/go-vagrant)

# go-vagrant
A golang wrapper around the Vagrant command-line utility.


## Installation
**go-vagrant** can be installed with `go get`:

```bash
go get github.com/bmatcuk/go-vagrant
```

Import it in your code:

```go
import "github.com/bmatcuk/go-vagrant"
```

The package name will be `vagrant`.


## Basic Usage
First, you'll need to instantiate a VagrantClient object using
`NewVagrantClient`. The function takes one argument: the path to the directory
where the Vagrantfile lives. This instantiation will check that the vagrant
command exists and that the Vagrantfile can be read.

From there, every vagrant action follows the same pattern: call the appropriate
method on the client object to retrieve a command object. The command object
has fields for any optional arguments. Then either call the command's `Run()`
method, or call `Start()` and then `Wait()` on it later. Any output from the
command, including errors, will be fields on the command object.

For example:

```go
package main

import "github.com/bmatcuk/go-vagrant"

func main() {
  client, err := vagrant.NewVagrantClient(".")
  if err != nil {
    ...
  }

  upcmd := client.Up()
  upcmd.Verbose = true
  if err := upcmd.Run(); err != nil {
    ...
  }
  if upcmd.Error != nil {
    ...
  }

  // TODO: vagrant VMs are up!
}
```


## Available Actions

### Destroy
Stop and delete all traces of the vagrant machines.

```go
func (*VagrantClient) Destroy() *DestroyCommand
```

Options:
* **Force** (default: `true`) - Destroy without confirmation. Defaults to true
  because, when it's false, vagrant will try to ask for confirmation but
  complain that there's no attached TTY.
* **Parallel** (default: `false`) - Enable or disable parallelism if provider
  supports it (automatically enables Force).

Response:
* **Error** - Set if an error occurred.


### Halt
Stops the vagrant machine.

```go
func (*VagrantClient) Halt() *HaltCommand
```

Options:
* **Force** (default: `false`) - Force shutdown (equivalent to pulling the
  power of the VM)

Response:
* **Error** - Set if an error occurred.


### Port
Get information about guest port forwarded mappings.

```go
func (*VagrantClient) Port() *PortCommand
```

Response:
* **ForwardedPorts** - a map of vagrant machine names to `ForwardedPort`
  objects. Each ForwardedPort has a `Guest` and a `Host` port, representing
  a mapping from the host port to the guest.
* **Error** - Set if an error occurred.


### Provision
Provision the vagrant machine.

```go
func (*VagrantClient) Provision() *ProvisionCommand
```

Options:
* **Provisioners** (default: `nil`) - Enable only certain provisioners, by type
  or name as an array of strings.

Response:
* **Error** - Set if an error occurred.


### Reload
Restarts the vagrant machine and loads any new Vagrantfile configuration.

```go
func (*VagrantClient) Reload() *ReloadCommand
```

Options:
* **Provisioning** (default: `DefaultProvisioning`) - By default will only run
  provisioners if they haven't been run already. If set to ForceProvisioning
  then provisioners will be forced to run; if set to DisableProvisioning then
  provisioners will be disabled.
* **Provisioners** (default: `nil`) - Enable only certain provisioners, by type
  or name as an array of strings. Implies ForceProvisioning.

Response:
* **Error** - Set if an error occurred.


### Resume
Resume a suspended vagrant machine

```go
func (*VagrantClient) Resume() *ResumeCommand
```

Options:
* **Provisioning** (default: `DefaultProvisioning`) - By default will only run
  provisioners if they haven't been run already. If set to ForceProvisioning
  then provisioners will be forced to run; if set to DisableProvisioning then
  provisioners will be disabled.
* **Provisioners** (default: `nil`) - Enable only certain provisioners, by type
  or name as an array of strings. Implies ForceProvisioning.

Response:
* **Error** - Set if an error occurred.


### SSHConfig
Get SSH configuration information for the vagrant machine.

```go
func (*VagrantClient) SSHConfig() *SSHConfigCommand
```

Options:
* **Host** (default: `""`) - Name the host for the config

Response:
* **Configs** - a map of vagrant machine names to `SSHConfig` objects. Each
  SSHConfig has several fields including Host, User, Port, etc. You can see
  full field list in the [godocs for SSHConfig].


### Status
Get the status of the vagrant machine.

```go
func (*VagrantClient) Status() *StatusCommand
```

Response:
* **Status** - a map of vagrant machine names to a string describing the
  status of the VM.
* **Error** - Set if an error occurred.


### Suspend
Suspends the vagrant machine.

```go
func (*VagrantClient) Suspend() *SuspendCommand
```

Response:
* **Error** - Set if an error occurred.


### Up
Starts and provisions the vagrant machine.

```go
func (*VagrantClient) Up() *UpCommand
```

Options:
* **Provisioning** (default: `DefaultProvisioning`) - By default will only run
  provisioners if they haven't been run already. If set to ForceProvisioning
  then provisioners will be forced to run; if set to DisableProvisioning then
  provisioners will be disabled.
* **Provisioners** (default: `nil`) - Enable only certain provisioners, by type
  or name as an array of strings. Implies ForceProvisioning.
* **DestroyOnError** (default: `true`) - Destroy machine if any fatal error
  happens.
* **Parallel** (default: `true`) - Enable or disable parallelism if provider
  supports it.
* **Provider** (default: `""`) - Back the machine with a specific provider.
* **InstallProvider** (default: `true`) - If possible, install the provider if
  it isn't installed.

Response:
* **VMInfo** - a map of vagrant machine names to `VMInfo` objects. Each VMInfo
  describes the `Name` of the machine and `Provider`.
* **Error** - Set if an error occurred.


### Version
Get the current and latest vagrant version.

```go
func (*VagrantClient) Version() *VersionCommand
```

Response:
* **InstalledVersion** - the version of vagrant installed
* **LatestVersion** - the latest version available
* **Error** - Set if an error occurred.


[godocs for SSHConfig]: https://godoc.org/github.com/bmatcuk/go-vagrant#SSHConfig
