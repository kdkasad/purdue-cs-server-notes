# Using Tailscale on the CS servers

It's possible to add the CS servers to your Tailscale network (Tailnet).
However, make sure you still follow the [acceptable use policies][aup] if you
choose to do so. Simply using Tailscale to connect to the servers from
off-campus does not appear to be a violation of the policies (according to my
understanding), but do this at your own risk.

[aup]: https://www.cs.purdue.edu/resources/aup.html

## Setup

To add a server to your Tailnet, you need to run `tailscaled` on the machine. We
can't do this as root, but `tailscaled` has a userspace networking mode in which
it doesn't need root access.

### Install Tailscale binaries

Install `tailscaled` and `tailscale` by downloading the static release binaries
from <https://pkgs.tailscale.com/stable/#static> onto the server (e.g. using
`curl` or `wget`). Then unpack the downloaded archive using `tar`. Copy the
`tailscaled` and `tailscale` binaries into `~/.local/bin`. Also add this
directory to your `PATH` so you can run the `tailscale` utility.

> [!NOTE]
> You can put them somewhere else, but you'll need to adjust the service file
> below to have the correct path to the binaries.

### Create the systemd service

To keep the daemon running, we'll use [Systemd], which manages running services
for us.

To create the service definition, run this command:
```sh
systemctl --user edit --full --force tailscaled.service
```

Then write the following into the editor and save the file:
```systemd
[Unit]
Description=Tailscale node agent
Documentation=https://tailscale.com/kb/
Wants=network-pre.target
After=network-pre.target
; After=NetworkManager.service systemd-resolved.service

[Service]
ExecStart=%h/.local/bin/tailscaled --statedir=${STATE_DIRECTORY} --socket=${RUNTIME_DIRECTORY}/tailscaled.sock --tun=userspace-networking
ExecStopPost=%h/.local/bin/tailscaled --statedir=${STATE_DIRECTORY} --socket=${RUNTIME_DIRECTORY}/tailscaled.sock --cleanup
Type=notify

Restart=on-failure

# Include hostname in state/cache directories because the filesystem is shared
# across machines.
RuntimeDirectory=tailscaled/%H
StateDirectory=tailscaled/%H
CacheDirectory=tailscaled/%H

[Install]
WantedBy=default.target
```

> [!NOTE]
> This configuration was developed by referencing both the Systemd manual pages
> and the Tailscale documentation. If you have questions about this service
> configuration, please try to reference those to answer your question. If you
> need more explanation, feel free to open an issue requesting elaboration and
> I'll do my best to explain how it works.

> [!IMPORTANT]
> The `--user` option to `systemctl` specifies that we want to interact with the
> `systemd` daemon running as our user account. If you leave that option out,
> `systemctl` will try to control the system daemon, which runs as `root`.

[Systemd]: https://systemd.io

### Running the service

To allow the service to stay running after you log out of the machine, we need
to enable lingering units:
```sh
loginctl enable-linger
```

Then we can enable the unit (which tells Systemd to start it automatically).
We'll also pass the `--now` flag to also start it right now instead of waiting
until the next login:
```sh
systemctl --user enable --now tailscaled.service
```

### Shell alias for `tailscale`

Since we're running `tailscaled` as our user and not as root, we're using
a different socket path from the default. The `tailscale` command assumes
`tailscaled` is being run as `root`, and thus that the socket is in the default
system-wide location.

To reconcile this, we must specify the `--socket` option to every invocation of
the `tailscale` command. To make this easier, we can define a shell alias that
does this automatically:
```sh
alias tailscale="tailscale --socket=\${XDG_RUNTIME_DIR}/tailscaled/$(hostname)/tailscaled.sock"
```
You can place this in your shellrc (`~/.bashrc`, `~/.zshrc`, etc.) to set this
alias by default every time you log in.

Verify that you've configured everything properly by running `tailscale status`.
It should print `Logged out.`. If instead you get a message like the following,
go back and check your work.
```
failed to connect to local tailscaled (which appears to be running as tailscaled, pid 1772571). Got error: Failed to connect to local Tailscale daemon for /localapi/v0/status; not running? Error: dial unix /some/socket/path: connect: no such file or directory
```

### Connecting to your Tailnet

Now that `tailscaled` is running and `tailscale` can control it, all that's left
to do is connect the daemon to your Tailscale account. Do this by running
`tailscale up` as you normally would. I won't provide specific options to use,
as that can depend on your Tailscale configuration and desired use cases.
