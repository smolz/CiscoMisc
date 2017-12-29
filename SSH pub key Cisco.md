# Using SSH public key authentication with Cisco

## Configuration

### Locate your public key

Find the file that contains your public key. Note that Cisco only supports ssh-rsa keys; if you are using a DSA key then you'll need to generate a new key pair.

If your public key is on a Linux box then it will probably be here:

```
$ cat ~/.ssh/id_rsa.pub
```
and it should start with "ssh-rsa"

### Wrap the key onto multiple lines

Cisco does not accept the entire key pasted into one long line, so you need to break it into multiple lines. The maximum line length at the Cisco CLI is 254 characters, but it's convenient to wrap smaller than this so that it fits on a terminal line.
```
$ fold -b -w 72 ~/.ssh/id_rsa.pub
```
Then copy this into your clipboard.

### Add your public key against the desired user

For example, if you have an existing user 'nsrc' and want to enable logins with your public key, enter the following configuration on your Cisco:
```
conf t
ip ssh pubkey-chain
username nsrc
key-string
<< paste your multi-line public key here >>
exit
```
Remember to hit Enter after the command key-string before pasting your key; and again after pasting your key before typing exit.

If you tried to use the wrong sort of key (e.g. a DSA key) you will see an error at this point.

### Verify

`show run | beg pubkey`

You should see a response like this:

```
ip ssh pubkey-chain
  username nsrc
   key-hash ssh-rsa EFF40492D1D6BF5D0B68491128456D27 yourname@yourdomain.example 
```
Note that the configuration doesn't show the actual public key, but its fingerprint instead. Compare this against the fingerprint of your public key to check you have the correct key loaded.
```
$ ssh-keygen -l -f ~/.ssh/id_rsa.pub
2048 ef:f4:04:92:d1:d6:bf:5d:0b:68:49:11:28:45:6d:27  yourname@yourdomain.example (RSA)
```
### Testing

You should now be able to login as the given user using your public key to authenticate - that is, you should no longer be prompted for the local password on the Cisco.

If it fails, and you are using an ssh agent for authentication, and you generated a new RSA key, the agent may not have unlocked (decrypted) the new key yet.

Under Linux, ssh-add -l will list all the keys your agent knows about and has unlocked. If your RSA key is not shown, then try the following commands to forget and then re-add all your keys to the agent:
```
$ ssh-add -d
$ ssh-add -a
```
## Additional information

### Security settings

You should disable the obsolete SSH version 1 protocol:

`ip ssh version 2`

And once you are sure that ssh is working correctly (with or without ssh key authentication) you most likely want to disable telnet entirely:
```
line vty 0 15
 transport input ssh
```
### Multiple keys

It is possible to have multiple public keys against the same user in the configuration, so that multiple people can login using the same account. Just add further keys as shown above. When you show the configuration, it will look like this:
```
ip ssh pubkey-chain
  username nsrc
   key-hash ssh-rsa EFF40492D1D6BF5D0B68491128456D27 yourname@yourdomain.example
   key-hash ssh-rsa 768CD77578B0C1B55BCC0C3549D3E573 another@yourdomain.example
```
### Removing passwords

You can remove passwords from users in the configuration; this will force those users to use ssh key authentication. If these people require admin rights they will either still need to know the enable secret, or their login can drop them directly into enable mode:

`username foo privilege 15`

### Disable passwords entirely

If you wish to disable password authentication entirely over ssh, but leave passwords on user accounts for other purposes (e.g. console access), then use:
```
no ip ssh server authenticate user password
no ip ssh server authenticate user keyboard
```
