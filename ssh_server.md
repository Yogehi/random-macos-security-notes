## Summary

MacOS has a built in SSH server. This is located at System Preferences -> Sharing

## Binary Location

"Keygen wrapper":
`/usr/libexec/sshd-keygen-wrapper`

Contents of keygen wrapper:
```
Yays-Mac:~ yayusernameyay$ cat /usr/libexec/sshd-keygen-wrapper
#!/bin/sh
SSHDIR=/etc/ssh

[ ! -f ${SSHDIR}/ssh_host_rsa_key ] && ssh-keygen -q -t rsa  -f ${SSHDIR}/ssh_host_rsa_key -N "" -C "" < /dev/null > /dev/null 2> /dev/null
[ ! -f ${SSHDIR}/ssh_host_dsa_key ] && ssh-keygen -q -t dsa  -f ${SSHDIR}/ssh_host_dsa_key -N "" -C "" < /dev/null > /dev/null 2> /dev/null
[ ! -f ${SSHDIR}/ssh_host_ecdsa_key ] && ssh-keygen -q -t ecdsa  -f ${SSHDIR}/ssh_host_ecdsa_key -N "" -C "" < /dev/null > /dev/null 2> /dev/null
[ ! -f ${SSHDIR}/ssh_host_ed25519_key ] && ssh-keygen -q -t ed25519  -f ${SSHDIR}/ssh_host_ed25519_key -N "" -C "" < /dev/null > /dev/null 2> /dev/null

exec /usr/sbin/sshd $@
```

OpenSSH binary location:
`/usr/sbin/sshd`

## Credential Confirmation Issue

This issue was first found on MacOS 10.13.4.

Lets say there's a MacOS computer with the users "ken" and "zak" on the computer, but only "ken" has ssh capabilities.

If you were to ssh in as "zak" and supply an incorrect password, the MacOS computer will take around 2 seconds to deny you.

If you were to ssh in as "zak" and supply a correct password, the MacOS computer will immediately close the ssh connection.

Note that any invalid login attempts via SSH does account against the user's lockout policy, and that if a user is locked out of their account then they cannot login via SSH.

## How SSH is started

As an admin user, go to System Preferences -> Sharing and enable Remote Login

The process "/sbin/launchd" will start listening on port 22:
`
lsof -nP +c 15 | grep LISTEN
<snip>
launchd           1            root   42u     IPv6 0xdcfa3871136ea603        0t0        TCP *:22 (LISTEN)
launchd           1            root   45u     IPv4 0xdcfa387116e83bab        0t0        TCP *:22 (LISTEN)
launchd           1            root   46u     IPv6 0xdcfa3871136ea603        0t0        TCP *:22 (LISTEN)
launchd           1            root   47u     IPv4 0xdcfa387116e83bab        0t0        TCP *:22 (LISTEN)`

Running `launchctl list | grep ssh` will show that `com.openssh.sshd` has been loaded:
`bash-3.2# launchctl list | grep ssh
-	0	com.openssh.sshd`

You can confirm this by (with sudo privs) unloading/loading the appropriate launch plist:
`bash-3.2# launchctl unload /System/Library/LaunchDaemons/ssh.plist
bash-3.2# launchctl load /System/Library/LaunchDaemons/ssh.plist`

Contents of `/System/Library/LaunchDaemons/ssh.plist`:
`
bash-3.2# cat /System/Library/LaunchDaemons/ssh.plist 
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>Disabled</key>
	<true/>
	<key>Label</key>
	<string>com.openssh.sshd</string>
	<key>Program</key>
	<string>/usr/libexec/sshd-keygen-wrapper</string>
	<key>ProgramArguments</key>
	<array>
		<string>/usr/sbin/sshd</string>
		<string>-i</string>
	</array>
	<key>Sockets</key>
	<dict>
		<key>Listeners</key>
		<dict>
			<key>SockServiceName</key>
			<string>ssh</string>
			<key>Bonjour</key>
			<array>
				<string>ssh</string>
				<string>sftp-ssh</string>
			</array>
		</dict>
	</dict>
	<key>inetdCompatibility</key>
	<dict>
		<key>Wait</key>
		<false/>
		<key>Instances</key>
		<integer>42</integer>
	</dict>
	<key>StandardErrorPath</key>
	<string>/dev/null</string>
	<key>SHAuthorizationRight</key>
	<string>system.preferences</string>
	<key>POSIXSpawnType</key>
	<string>Interactive</string>
</dict>
</plist>`