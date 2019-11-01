## Securing Access to Routers

### Design

> Why do we need to configure this? What need to be set up? What component is involved in the design (router, laptop, etc.) ? ...

Accessing the routers through the physical "console" port is inconvenient and dangerous. Thus, remote acess through Secure Shell (SSH) protocol to routers is needed. 

In our network, we enable remote SSH access on all 3 routers. We use seperate combinations of username and password on each router to ensure the independence of security of each router.

In addition, we set up SSH public key authentication on one of our laptops (BT001), which allows the user on the laptop to login in to all routers without entering passwords.



### Implementation

> Which cable is used? Which components is the cable connected to? What commands are typed in? ...

We first set up remote SSH access as instructed in Reference Guide on all 3 routers. Below is the configuration commands for Router 1 (BT-R001).

```
hostname BT-R001
ip domain name bt.lboro
username r001 priv 15 secret <secret>
line vty 0 4
transport input ssh telnet
login local

ip ssh version 2
crypto key generate rsa general-keys
ip ssh dh min size 4096
```

We then generate a pair of public and private keys on Laptop 1 (BT001).

```
ssh-keygen
```

After that, the pair of keys is written into file `~/.ssh/id_rsa.pub` and  `~/.ssh/id_rsa.pub` . We use the generated public key ( `id_rsa.pub` ) to set up SSH public key authentication on all 3 routers.

```
ip ssh pubkey-chain
username r001
key-string
```



### Evaluation

> How do we now it is successfully implemented? What happens if it fails? ...

Once remote SSH access is set up on 3 routers, we should be able to access them on Laptop 1 (BT001) without entering the password using the following commands.

```sh
# access Router 1
ssh r001@23.0.0.1 
# access Router 2
ssh r002@23.0.0.50
# access Router 3
ssh r003@23.0.0.33
```

Screenshots of successful remote access to all 3 routers are shown in Figure 1, Figure 2 and Figure 3 respectively.



<img src="Figures/ssh1.jpg" alt="ssh1" style="zoom:10%;" />

> Figure 1. Sucessful remote SSH access to Router 1 (BT-R001).



<img src="Figures/ssh2.jpg" alt="ssh2" style="zoom:10%;" />

> Figure 2. Sucessful remote SSH access to Router 2 (BT-R002).



<img src="Figures/ssh3.jpg" alt="ssh3" style="zoom:10%;" />

> Figure 3. Sucessful remote SSH access to Router 3 (BT-R003).



### Commentary

#### Problem: Maximum Limit of Characters per Line

When we tried to set up SSH public key authentication on routers, we failed at our initial attempt. It turned out that Cisco router has maximum limit of characters for each command line. Thus, a public key in a single long line were not accepted by the router. 

To solve this problem, we then used `fold` to split the public key into multiple lines before re-uploading the key and SSH public key authentication was successfully set up on the router.

