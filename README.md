# OpenVPN Server Role

The OpenVPN Server Role install and configure OpenVPN with help of Ansible. The Server role should be run in combination with the [OpenVPN Client Role](https://github.com/tieto-cem/openvpn-client-role).

## Getting started with the OpenVPN roles

### Requirements

The OpenVPN module requires that an Ubuntu machine is running with a public IP-address.

For AWS there is an [OpenVPN terraform module](https://github.com/tieto-cem/terraform-aws-openvpn) that provision an Ubuntu machine in the public subnet of the VPC.

- [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) is requred in order to use the OpenVPN roles
- SSH key pair

### Install OpenVPN on the Server

Ansible automatically install OpenVPN with all dependencies and configurations.

#### Ansible playbook

Run the following commands in order to install the server role and the client role

```bash
ansible-galaxy install git+https://github.com/tieto-cem/openvpn-server-role.git,v1.2.0
ansible-galaxy install git+https://github.com/tieto-cem/openvpn-client-role.git,v1.2.0
```

Create the playbook, e.g. with this [example](example/). The project structure for the example look like this:

```console
.
├── inventory
|   └── project_inventory
├── group_vars
|   └── project_vars
├── install-openvpn.yml
├── install.sh
└── obtain-keys.sh
```

Redefine the inventory file, [project_inventory](example/inventory/project_inventory), use the public IP address of the Ubuntu machine. Also modify the location to the projects private SSH key.

```bash
[example-project]
34.100.10.1

[all:vars]
ansible_connection=ssh
ansible_ssh_private_key_file=~/.ssh/example-project.pem
ansible_ssh_user=ubuntu
ansible_python_interpreter=/usr/bin/python3.5
```

Redefine the group variables in the group_vars file, [project_vars](example/group_vars/project_vars).

| Variable | Description |
| --- | --- |
| VpnClients | List of VPN keys that should be generated |
| ServerName | Name of the server, will be part of keys etc |
| RedrirectTraffic | If `true` all traffic will get routed trough the VPN server. Normally this can be `false` |
| ServerNetwork | Network reserved for the server |
| LocalNetworks | Clients will route all traffic trough the VPN server within these IP-ranges |
| DhcpOptions | Optional - Allows server to push DHCP options (like nameserver) to client |
| NetworkInterface | Network interface for the public IP |

#### Run ansible

Run the playbook in order to install OpenVPN and create user keys (you can also use the `install.sh` in the example directory).

```bash
ANSIBLE_HOST_KEY_CHECKING=false ansible-playbook install-openvpn.yml -i inventory
```

#### Obtain client keys

The client keys exist on the server and can only be obtain with sudo rights.

Run the following command to obtain client keys (`obtain-keys.sh` is located in the example directory)

```bash
./obtain-keys.sh -i /location/to/private/example-project.pem -s ubuntu@IP_ADDRESS -c username
```

### Client installation for OpenVPN

#### Mac OS Client

Install stable version of [Tunnelblick](https://tunnelblick.net/downloads.html).

Extract your key file (your-name.tar.gz) into an empty directory e.g. vpn. Run the commands in a terminal window

```bash
mkdir ~/Documents/vpn && cd ~/Documents/vpn
tar zxf /location/to/your-name.tar.gz
```

Open Finder and locate the folder `~/Documents/vpn`.

Double click on the the conf file e.g. cem-openvpn-your-name.conf and choose to install for "Only Me".

Click on the small Tunnelblick icon in the top menu bar and choose "Connect cem-openvpn-your-name".

#### Linux Client (Ubuntu 16.04 example)

Install OpenVPN

```bash
wget -O - https://swupdate.openvpn.net/repos/repo-public.gpg | sudo apt-key add -
echo "deb http://build.openvpn.net/debian/openvpn/release/2.4 xenial main" | sudo tee /etc/apt/sources.list.d/openvpn-aptrepo.list
sudo apt update && sudo apt install openvpn
```

Extract your key file (your-name.tar.gz) into `/etc/openvpn`.

```bash
cd /etc/openvpn/
sudo tar zxf /location/to/your-name.tar.gz
```

Start OpenVPN client

```bash
sudo systemctl status openvpn@cem-openvpn-your-name
```

Stop OpenVPN client

```bash
sudo systemctl stop openvpn@cem-openvpn-your-name
```

#### Windows Client

Install latest version of [OpenVPN](https://openvpn.net/index.php/download/community-downloads.html) for Windows.

Extract Extract your key file (your-name.tar.gz) into `C:\Program Files\OpenVPN\config`. WinZip can be used for extracting .tar.gz files.

Right click on the small OpenVPN icon in the windows taskbar on the right hand bottom corner and select connect.

## License

MIT © [Tieto CEM](https://www.tieto.com/en/what-we-do/digital-experience-and-consulting/customer-experience-management/)
