# Setup a tacacs+ server on Ubuntu 16.04 LTS

## Setup tacacs+ to use local accounts on the ubuntu server

* ### Create local users - example below
  ```
  admin@ubuntu:~$ sudo adduser engineer
  Adding user `engineer' ...
  Adding new group `engineer' (1001) ...
  Adding new user `engineer' (1001) with group `engineer' ...
  Enter new UNIX password:
  Retype new UNIX password:
  passwd: password updated successfully
  Changing the user information for engineer
  Enter the new value, or press ENTER for the default
   Full Name []:
   Room Number []:
   Work Phone []:
   Home Phone []:
   Other []:
  Is the information correct? [Y/n] y
  ```
* ### Download and install TACACS+
  `sudo apt-get install tacacs+`

* ### Edit the config file with nano - _Below are the parts to change_
  The default service = permit parameter tells TACACS+ daemon that all commands are allowed for this group. 
  The login = file /etc/passwd parameter tells the daemon to look for the user account and password matches in the file. 
  If it matches, allow the account to log in to the router and switch. 
  The enable = file /etc/passwd tells the daemon that it needs to match the password of the user account. 
  If it matches, allow to enter privileged EXEC mode, also known as enable mode.

  The default service = deny parameter which tells the daemon that all other commands except for user EXEC commands are allowed. 
  You will need to permit the commands that they’re allowed to use. 
  Service = exec and priv-lvl = 2 allows us to give a higher privilege than an ordinary user. 
  We do not want to give this group a privilege level of 15, meaning the same level as the Network Admins.
  `sudo nano /etc/tacacs+/tac_plus.conf`

  ```
  accounting file = /var/log/tac_plus.acct
  key = testing123
  user = engineer {
    member = Network_Admins
  }
  user = manager {
    member = network_operators
  }

  group = Network_Admins {
    default service = permit
    login = file /etc/passwd
    enable = file /etc/passwd
    service = exec {
    priv-lvl = 15
    }
  }

  group = network_operators {
    default service = deny
    login = file /etc/passwd
    enable = file /etc/passwd
    service = exec {
    priv-lvl = 2
    }
    cmd = enable {
      permit .*
    }
    cmd = show {
      permit .*
    }
    cmd = exit {
      permit .*
    }
  }
  ```
  
* ### Restart tac_plus daemon
  `sudo service tacacs+ restart`
    
* ### Configure Cisco device
  ```
  aaa group server tacacs+ tac_plus
   server-private x.x.x.x key TACACS_KEY
   ip vrf forwarding mgmt

  aaa authentication login default group tac_plus local
  aaa authentication enable default group tac_plus enable
  aaa authorization config-commands
  aaa authorization exec default group tac_plus if-authenticated 
  aaa authorization commands 15 default group tac_plus if-authenticated 
  !
  ```
    
