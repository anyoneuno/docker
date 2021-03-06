# Ansible on Apline

Ansible installation on Alpine 3.4. Based on [williamyey/ansible](https://hub.docker.com/r/williamyeh/ansible/). Includes support for provisioning Windows machines.

## Remote System Requirements

The remote system must be prepared according to the type:

* [Windows](http://docs.ansible.com/ansible/intro_windows.html#windows-system-prep)
* [SSH](http://docs.ansible.com/ansible/intro_getting_started.html#remote-connection-information)
* Docker - see [Dockerfile.sshd](Dockerfile.sshd) for an example minimal setup

## Example Ansible Configuration

* [ansible.cfg](ansible.cfg) - Ansible global configuration parameters
* [hosts](hosts) - lists of hosts to operate against
* [windows.yml](group_vars/windows.yml) - configuration specific to the `windows` group in [hosts](hosts/#L1)
* [docker.yml](group_vars/docker.yml) - configuration specific to the `docker` group in [hosts](hosts/#L3)

## Local Ansible Build

### Local Image

From the directory containing the Dockerfile.local:

```bash
docker build -t ansible -f Dockerfile.local .
```

### Local Test

Use the `ansible --version` command to perform a simple test which displays the Ansible version in the container: 

```bash
$ docker run --rm -it ansible ansible --version
ansible 2.1.1.0
  config file = 
  configured module search path = Default w/o overrides
$
```

## Docker Test Target
A test target in Docker may be used for further provisioning tests and will be used here to test the [Local Image](#local-image)

### Test Target Build
The Docker test target image is built using [Dockerfile.sshd](Dockerfile.sshd):

```bash
docker build -t sshd -f Dockerfile.sshd .
```

### Test Target Run
The test target container may then be started in detached mode:

```bash
docker run -d --name sshd -p 22 sshd
```

### Linked Ping Test
The test target is then linked (`--link sshd`) to a Ansible container to run a simple ping command (`ansible docker -m ping`) using Ansible and the [docker](hosts/#L3) group. 

```bash
$ docker run --rm -it --link sshd ansible ansible docker -m ping
sshd | SUCCESS => {
    "changed": false, 
    "ping": "pong"
}
$
```

## Executing Commands on Windows

### Ping the Windows Box

The `ansible <inventory> -m win-ping` command may be used to attempt to ping the target.

```bash
$ docker run --rm -it ansible ansible windows -m win_ping
10.10.101.173 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
$
```

### Raw Commands

The `raw` module may be used to issue raw commands (like `DIR` to list the contents of the remote directory):

```bash
$ docker run --rm -it ansible ansible windows -m raw -a "CMD /C dir"
10.10.101.173 | SUCCESS | rc=0 >>
 Volume in drive C is Windows 10
 Volume Serial Number is 905D-2DA8

 Directory of C:\Users\Administrator

08/12/2016  11:51 AM    <DIR>          .
08/12/2016  11:51 AM    <DIR>          ..
08/12/2016  11:51 AM    <DIR>          .ansible
08/12/2016  12:18 PM                10 .bash_history
07/16/2016  04:47 AM    <DIR>          Desktop
08/12/2016  11:45 AM    <DIR>          Documents
07/16/2016  04:47 AM    <DIR>          Downloads
07/16/2016  04:47 AM    <DIR>          Favorites
07/16/2016  04:47 AM    <DIR>          Links
07/16/2016  04:47 AM    <DIR>          Music
07/16/2016  04:47 AM    <DIR>          Pictures
07/16/2016  04:47 AM    <DIR>          Saved Games
07/16/2016  04:47 AM    <DIR>          Videos
               1 File(s)             10 bytes
              12 Dir(s)  29,668,818,944 bytes free


$
```

### Other Modules

Refer to [Windows Modules](http://docs.ansible.com/ansible/list_of_windows_modules.html) for a listing an description of other available modules.
