### Automate the Provisioning and Configuration of HAProxy and an Apache Web Server Cluster Using Foreman
Use Vagrant, Foreman, and Puppet to provision and configure HAProxy as a reverse proxy, load-balancer for a cluster of Apache web servers. Project is part of my blog post, [Automate the Provisioning and Configuration of HAProxy and an Apache Web Server Cluster Using Foreman](http://wp.me/p1RD28-1ok).

The GitHub project has been updated 8/23/2015. Changes were required to fix incapability issues with the latest versions of Puppet and Foreman. See http://theforeman.org/manuals/1.9/index.html#3.1.2PuppetCompatibility for details. Additionally, the version of CentOS on all VMs was updated from 6.6 to 7.1 and the version of Foreman was updated from 1.7 to 1.9.

<p><a href="https://programmaticponderings.wordpress.com/?attachment_id=3459" title="New Foreman Hosts" rel="attachment"><img width="620" height="390" src="https://programmaticponderings.files.wordpress.com/2015/08/new-foreman-hosts.png?w=620" alt="New Foreman Hosts"></a></p>

#### Vagrant Plug-ins
This project requires the Vagrant vagrant-hostmanager plugin to be installed. The Vagrantfile uses the vagrant-hostmanager plugin to automatically ensure all DNS entries are consistent between guests as well as the host, in the `/etc/hosts` file. An example of the modified `/etc/hosts` file is shown below.
```text
## vagrant-hostmanager-start id: c472843a-e854-4e58-8a13-856b3b0766f2
192.168.35.5   theforeman.example.com
192.168.35.101 haproxy.example.com
192.168.35.121 node01.example.com
192.168.35.122 node02.example.com
## vagrant-hostmanager-end
```

You can manually run `vagrant hostmanager` to update `/etc/hosts` at anytime.  

This project also requires the Vagrant vagrant-vbguest plugin is also used to keep the vbguest tools updated.
```sh
vagrant plugin install vagrant-hostmanager
vagrant plugin install vagrant-vbguest
```

#### JSON Configuration File
The `Vagrantfile` retrieves multiple VM configurations from a separate `nodes.json` JSON file. All VM configuration is
contained in that JSON file. You can add additional VMs to the JSON file, following the existing pattern. The
`Vagrantfile` will loop through all nodes (VMs) in the `nodes.json` file and create the VMs. You can easily swap
configuration files for alternate environments since the `Vagrantfile` is designed to be generic and portable.


#### Instructions
Provision the Foreman VM first, before the agents. It will takes several minutes to fully provision the VM.
```sh
vagrant up theforeman.example.com
```
Important, when the provisioning is complete, note the output from Vagrant. The output provides the `admin` login password and URL for the Foreman console. Example output below.
```text
==> theforeman.example.com:   Success!
==> theforeman.example.com:   * Foreman is running at https://theforeman.example.com
==> theforeman.example.com:       Initial credentials are admin / 7x2fpZBWgVEHvzTw
==> theforeman.example.com:   * Foreman Proxy is running at https://theforeman.example.com:8443
==> theforeman.example.com:   * Puppetmaster is running at port 8140
==> theforeman.example.com:   The full log is at /var/log/foreman-installer/foreman-installer.log
```
Log into Foreman's browser-based console using the information provided in the output from Vagrant (example above). Change the `admin` account password, and/or set-up your own `admin` account(s).

Next, build three puppet haproxy and apache VMs. Again, it will takes several minutes to fully provision the two VMs.
```sh
vagrant up
```

Next, complete the CSR process. Read the [blog post](http://wp.me/p1RD28-1ok) for complete instructions.
```sh
# ssh into haproxy node
vagrant ssh haproxy.example.com
# initiate certificate signing request (CSR)
sudo puppet agent --test --waitforcert=60
# sign certificate within foreman to complete CSR
```

```sh
# ssh into first node
vagrant ssh node01.example.com
# initiate certificate signing request (CSR)
sudo puppet agent --test --waitforcert=60
# sign certificate within foreman to complete CSR
```
  
```sh
exit
# ssh into second node
vagrant ssh node02.example.com
# initiate certificate signing request (CSR)
sudo puppet agent --test --waitforcert=60
# sign certificate within foreman to complete CSR
```

#### Forwarding Ports
To expose forwarding ports, add them to the 'ports' array. For example:

 ```JSON
 "ports": [
        {
          ":host": 1234,
          ":guest": 2234,
          ":id": "port-1"
        },
        {
          ":host": 5678,
          ":guest": 6789,
          ":id": "port-2"
        }
      ]
```

#### Errors
**Error: Unknown configuration section 'hostmanager'.**
=> **Solution: **Install the `vagrant-hostmanager` plugin with `vagrant plugin install vagrant-hostmanager`

#### Useful Multi-VM Commands
The use of the specific <machine> name is optional in most cases.
* `vagrant up <machine>`
* `vagrant reload <machine>`
* `vagrant destroy -f <machine> && vagrant up <machine>`
* `vagrant status <machine>`
* `vagrant ssh <machine>`
* `vagrant global-status`

#### Useful Logs for Debugging Project Issues
Some logs require sudo access
* `sudo tail -50 /var/log/syslog`
* `sudo tail -50 /var/log/puppet/masterhttp.log`
* `sudo tail -50 /var/log/foreman/production.log`
* `sudo tail -50 /var/log/foreman-installer/foreman-installer.log`
* `sudo tail -50 /var/log/foreman-proxy/proxy.log`
* `tail -50 ~/VirtualBox\ VMs/<machine>/Logs/VBox.log`