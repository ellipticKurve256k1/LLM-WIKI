### ssh config 

```
Host $bastion_name
	Hostname $ip_addr
	User $user
	IdentityFile ~/.ssh/$keypair.pem
	IdentitiesOnly yes
	ServerAliveInterval 60
```

### ProxyJump

```
Host $private_name
	Hostname $ip_addr
	User $user
	IdentityFile ~/.ssh/$keypair.pem
	IdentitiesOnly yes
	ServerAliveInterval 60
	ProxyJump $bastion_name 
```

#### ssh into $private_name

```ssh
ssh $private_name
```

### `known hosts`

`~/.ssh/known_hosts` records the **identity of SSH servers you have connected to before**. 

```sh
172.31.0.45 ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAA...
```

it can be manually deleted to avoid the conflict