# Sharing abduco sessions

Some script to share abduco sessions with socat and SSH.

## Client setup

Clone or download this repository.

| Dependency  | Viewing lectures | Giving lectures | Pair programming |
|-------------|------------------|-----------------|------------------|
| SSH & shell | required         | required        | required         |
| abduco      |                  | required        | required         |
| socat       |                  | required        | required         |
| dvtm        |                  | required        | required on host |
| uuidgen     |                  |                 | required on host |
| not Windows |                  | required        | required         |

### Installation links (or use your package manager)

- [socat](http://www.dest-unreach.org/socat/)
- [abduco](https://github.com/martanne/abduco)
- [dvtm](https://github.com/martanne/dvtm)
- uuidgen: in your util-linux package, or something similar
- [Not windows](https://search.snopyta.org/?q=install+linux)

## Usage

### Viewing a lecture

Before you start: press the space bar to leave. Run `./view <lecture>`, replacing `<lecture>` with the lecture name your lecturer gave you. You should get a view of the terminal emulator of your lecturer in yours.

**On windows?** The script won't work for you, but if you have SSH on your computer (and you probably do), just running the command on the second line in the `view` script in a large command prompt window should work.

### Pair programming

For one-on-one pair programming. The host should run the `host` script. The script will print out a session ID which should be shared with the guest. Then, it will open a new shell which is controlled by both the host and the guest. Yes, there may be remote code execution. Exit the shell to stop hosting.

```sh
$ ./host
Session ID: e989a26d-17dd-4d8e-bad3-43074f813148
```

The guest receives a session ID from the host and passes this to the `guest` script. Press `Ctrl+\` to leave (or exit the shell, or shut down their computer).

```sh
$ ./guest e989a26d-17dd-4d8e-bad3-43074f813148
```

### Giving a lecture

Run `./share <lecture>`, replacing `<lecture>` with a nice name for your lecture (alphanumeric and - allowed).

Share the name of your lecture with your students so they can get a view of your terminal emulator. Exit the shell to stop sharing.

## Server setup

Lecture viewing and pair programming require the same setup, with a different script. Do this twice (with a different user) if you want both.

Choose a username, e.g. pairing. Append to `/etc/ssh/sshd_config` the lines below:

```
PrintLastLog no
Match User pairing
	AllowTcpForwarding no
	X11Forwarding no
	PermitTunnel no
	GatewayPorts no
	AllowAgentForwarding no
	PasswordAuthentication yes
	PermitEmptyPasswords yes
```

Also comment the `AcceptEnv` line to avoid locale warnings and add `pairing` to the `AllowUsers` list.

If using pam, modify `/etc/pam.d/common-auth` to change `nullok_secure` to `nullok` to permit empty passwords with pam (as per https://superuser.com/questions/1152645/openssh-server-anonymous-account). Modify `/etc/pam.d/sshd` to comment out the two motd lines.

Create the user with the `server/viewing` (for lecture viewing) or `server/pairing` (for pair programming) script as shell and an empty password:

```sh
sudo useradd -m -s /home/pairing/shell pairing
sudo su -s /bin/sh pairing
cp <the script> shell
chmod 500 shell
exit
sudo passwd -d pairing
```

Fork this repository and change the `SERVER` constant in the scripts (or instruct your student to change it).
