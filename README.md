# AutoSSH init.d script

autossh-init - AutoSSH init script for starting a reverse SSH tunnel as a service.

I want to backup my remote server via SSH/rsync to my home Linux server which is behind a NAT, and has a private IP address. Port forwarding could be used but my home public IP can change at any time. With AutoSSH I can instead let my local home server connect to the web server I want to backup, and have it connect back to the home server creating a SSH-tunnel.

AutoSSH will also monitor the connectivity of the tunnel and re-establish the connection if dropped.

## Usage

### Create a restricted user

Add a restricted user on the **local and remote** machine:

    $ adduser --system --group --shell /bin/false \
      --home /var/lib/autossh --disabled-password autossh


### Generate SSH key

Generate a ssh key for the autossh user on the **local** machine. The key `<<PUBKEY>>` below is the contents of the generated file */var/lib/autossh/.ssh/id_rsa.pub*:

    $ su -s /bin/bash -c /usr/bin/ssh-keygen autossh


### Add SSH key to authorized keys

Add the `<<PUBKEY>>` to the authorized keys file on the **remote** machine to allow passwordless authentication:

	$ su -s /bin/bash -c 'echo <<PUBKEY>> >> ~/.ssh/authorized_keys' autossh
    
Optionally if more security is required one could restrict so no shell can be started:

    $ su -s /bin/bash -c \
    'echo "command=\"/bin/false\",no-agent-forwarding,no-pty,no-X11-forwarding <<PUBKEY>>" >> ~/.ssh/authorized_keys' autossh

Even more security can be achieved allowing only connections from certain hosts:

    $ su -s /bin/bash -c \
    'echo "command=\"/bin/false\",no-agent-forwarding,no-pty,no-X11-forwarding,permitopen=\"host1:port1\", permitopen=\"host2:port2\" <<PUBKEY>>" >> ~/.ssh/authorized_keys' autossh
    
    
### Verify passwordless authentication

On the **local** machine, try to connect to the remote server and confirm the fingerprint

	$ su -s /bin/bash -c 'ssh autossh@remote FAIL' autossh

        
### Install the service

On the **local** machine, copy the autossh.init-script and the configuration.

    $ cp -i autossh.init /etc/init.d/autossh
    $ chmod +x /etc/init.d/autossh
    $ update-rc.d autossh defaults

    $ cp -i autossh.default /etc/default/autossh    
    $ service autossh start

### Change the configuration

Change the configuration on the **local** machine to suit your needs and restart the service. Examples can be found in [autostart.default]:

    $ nano /etc/default/autossh    
    $ service autossh restart
    

## Author

Original work by [Felix C. Stegerman](mailto:flx@obfusk.net)

Updated documentation and examples by Joacim Breiler

## License

Copyright (C) 2013 Felix C. Stegerman. Released under [GPLv2 License](https://www.gnu.org/licenses/old-licenses/gpl-2.0.txt).


