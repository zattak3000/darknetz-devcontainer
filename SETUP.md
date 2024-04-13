# Development Setup

This repository takes advantage of the Visual Studio Code tasks feature as well as Development containers for quick setup and development, but neither are required.

## 1. Devcontainer setup

From [containers.dev](https://containers.dev/):

> "A development container (or dev container for short) allows you to use a container as a full-featured development environment. It can be used to run an application, to separate tools, libraries, or runtimes needed for working with a codebase, and to aid in continuous integration and testing."

While Visual Studio Code works best for devcontainers in my opinion, **it is not required by any means**. There are a [number of other IDEs and tools](https://containers.dev/supporting) that support the standard.

### Using Visual Studio Code

Install the Dev Containers extension by searching it in the extensions tab. Or, press Ctrl+P and enter
```text
ext install ms-vscode-remote.remote-containers
```

After cloning the repository, simply click the "Reopen in Container" button that appears in the bottom right popup to automatically build and enter the container.

### Using [Devcontainer CLI](https://github.com/devcontainers/cli)

Install with NPM

```bash
npm install -g @devcontainers/cli
```

Bring up the container (after cloning the repository)

```bash
devcontainer up --workspace-folder .
```

Run `make` inside the container

```bash
devcontainer exec --workspace-folder . make
```


### Using Docker only
Since this devcontainer is fairly simple, you could get away simply building the dockerfile (or pulling the image) and running the container interactively with a bind mount

After cloning the repository:
```bash
docker build -t darknetz-devcontainer .devcontainer/
```

Then run the container to run `make`:

```
docker run -dt --rm \
--mount type=bind,src=$(pwd),dest=/optee/darknetz \
darknetz-devcontainer
```

## 2. Pi Setup
To allow remote debugging to work, your computer must be able to access SSH and gdbserver on the Pi over the Ethernet port. This can be done a number of ways, but my suggestions are below.

You can modify these files while the Pi is running over the serial port, or mount the SD card on your computer and make changes that way.

### Static IP

The easiest way I found to do this was to assign a static address in the `/etc/network/interfaces` file.

The address can really be anything, but if you are working on windows the APIPA address space is very convenient when connecting the Pi over Ethernet directly to your computer.

`/etc/network/interfaces` file on Pi:

```plain
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
    address 169.254.100.100
    netmask 255.255.0.0
```

### Fix network init script

The `/etc/init.d/S40network` script attempts to bring up all network interfaces before the `lan78xx` kernel driver sets up the ethernet device, this is enough to fix it.

Add these lines to the top of `/etc/init.d/S40network`:

```bash
while [ ! -e /sys/class/net/eth0 ]; do
        echo "Waiting for eth0 to come up"
        sleep 1
done
```

### Disable `dhcpcd` (Optional)

I'm not *totally* sure if this is required, but sometimes the Pi's Ethernet interface doesn't come up after booting. I don't know if this is caused by `dhcpcd` trying to override the static address but since we aren't using DHCP in this setup I just turn it off to be safe.

I just rename it to something that doesn't start with S to keep the file but disable the service, since that's how the init system works.

```bash
mv /etc/init.d/S41dhcpcd /etc/init.d/OFF.S41dhcpcd
```

### Enable SSH Login

> [!CAUTION]
> I only suggest doing this if you are connecting the Pi directly to your PC. Allowing empty password SSH logins is generally not a good idea.

Add these lines to the top of `/etc/ssh/sshd_config`:

```text
PermitRootLogin yes
PasswordAuthentication yes
PermitEmptyPasswords yes
```

## 3. Pi Testing

Now that everything is setup, it is time to build and test on the Pi.

### Run `make`

If using Visual Studio Code, make the binaries by clicking **Terminal>Run Task>Make**.

If you are not using Visual Studio Code, go back to the first section to see how to execute make using alternative setups

### Upload to the Pi

Firstly, configure the `ssh_config` file in the root of the repository with your Pi's IP address and port under the `rpi3` host

Again, if using Visual Studio Code you can upload to the pi using the **Upload To Pi over SSH** task

If not, these commands will do the same thing:

```bash
scp -F ssh_config host/optee_example_darknetp rpi3:/usr/bin/darknetp
ssh -F ssh_config rpi3 chmod +x /usr/bin/darknetp
scp -F ssh_config ta/7fc5c039-0542-4ee1-80af-b4eab2f1998d.ta rpi3:/lib/optee_armtz/7fc5c039-0542-4ee1-80af-b4eab2f1998d.ta
ssh -F ssh_config rpi3 chmod 444 /lib/optee_armtz/7fc5c039-0542-4ee1-80af-b4eab2f1998d.ta
```

### Start `gdbserver`
> [!NOTE]
> gdbserver will not automatically load new versions of darknetz if uploaded while gdbserver is running. While there are commands to reload it, the easiest way is to simply restart gdbserver after uploading a new build.

Start gdbserver on port 12345 on the Pi by SSH-ing to it and running

```bash
gdbserver :12345 /usr/bin/darknetp <args>
```

Visual Studio Code has a very nice debugging interface which has been configured to connect to `gdbserver`. Press F5 after setting some breakpoints in the UI to use it.

If you would rather use regular gdb:
```bash
gdb-multiarch -ex "target remote :12345"
```