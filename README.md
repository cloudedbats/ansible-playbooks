# Ansible for Raspberry Pi bat detectors

Ansible is a command-line application used to configure systems and deploy software,
mainly used on clusters of server computers.
Ansible uses YAML files to define playbooks that describes the installation process,
and it uses a declarative format much easier to understand than ordinary shell scripts.

When using Ansible you get a well documented installation process as well as a
documentation of the specific configuration parameters for each used computer.

Ansible uses **playbooks** and **inventories** to define the workflows and to list
the computer nodes that should be managed.

Ansible should be run from a separate Linux/Unix computer
(or inside a virtual Linux computer, macOS also works)
and it should be possible to access all other computers via ssh. You can run the same
playbook multiple times, Ansible is smart enough to know what is already installed.

To make it simple there are directories here containing one playbook and one example
inventory file. It is recommended to copy that inventory file to /etc/ansible/hosts
and store your private settings away from the git controlled directories.

Documentation: https://docs.ansible.com/ansible_community.html

## Installation of CloudedBats - WURB 2020

Main workflow, overview:

- Install the Raspberry Pi Operating system on an SD card.
- Move the SD card to a Raspberry Pi computer and power it up.
- Download the Ansible playbook and inventory files into another Linux/Unix computer.
- Adjust the inventory file and run Ansible.
- Restart the Raspberry Pi.
- Add extra features not managed by Ansible, like RaspAP, etc.

## Install the Raspberry Pi Operating system

Use the **Raspberry Pi Imager** to install the Raspberry Pi operating system.

- Download and install the Raspberry Pi Imager from here:
https://www.raspberrypi.com/software/
- Start Raspberry Pi Imager.
- Select the operating system "Raspberry Pi OS Lite", use the 32-bit version.
- Attach an SD card and select it.
- Click the “settings” button (the cogwheel).

Then make these setting, to be used as an example. It will also work with an 
Ethernet cable and then the WiFi part can be omitted.

Note that the username must be **pi** in this version of the WURB bat detector.

- Hostname: wurb99
- Enable SSH
- Use password authentication
- Username: pi
- Password: my-wurb99-password
- Configure wireless LAN
  - SSID: my-home-wifi
  - Password: my-home-wifi-password
  - Wireless LAN country: SE
- Locale settings
  - Time zone: Europe/Stockholm
  - Keyboard layout: se

Finally write to the SD card. When finished move the SD card to the Raspberry Pi.

## Install Ansible

Note: This should be installed on another Linux or Unix computer,
not on the same Raspberry Pi that is used as a bat detector.
Since Ansible is a Python application it is installed with pip.

    git clone https://github.com/cloudedbats/ansible-playbooks.git
    cd ansible/
    python3 -m venv venv
    source venv/bin/activate
    pip install -r requirements.txt 

## Generate SSH keys and distribute them

SSH keys is the easiest way to run Ansible.
It is also a secure and easy way to log in to other computers even if they have
different passwords.

Generate SSH keys if that not already done.
More info here:
https://www.raspberrypi.com/documentation/computers/remote-access.html#generate-new-ssh-keys

The command to generate them is. You don't have to specify a passphrase, but it is recommended.

    ssh-keygen

And then the public key should be copied to each target computer. You need to know the
password for the target computer when establishing the connection.

    ssh-copy-id pi@wurb99.local

## Run Ansible

Go to the directory where the playbook is located. Then execute the Ansible playbook on all
connected computers that are described in the inventory list.

    cd raspberrypi_wurb_2020

    # Use this if the inventory list is copied to /etc/ansible/hosts:
    ansible-playbook -v playbook.yaml

    # Use this if the template inventory list is modified:
    ansible-playbook -i inventory.yaml -v playbook.yaml

Some other useful Ansible commands

    # Ordinary unix commands can be executed via ansible, 
    # for example "df -h" for all computers in the host group "wurbs".
    ansible -i inventory.yaml wurbs -a "df -h"

    # Strict format control when writing playbooks.
    ansible-lint playbook.yaml

    # Check playbook execution without running it.
    ansible-playbook -i inventory.yaml -v playbook.yaml --check

    # Limit the playbook execution to a single target computer.
    ansible-playbook -i inventory.yaml -v playbook.yaml --limit "wurb99"

## Restart the Raspberry Pi bat detector

This can be done by using Ansible

    ansible -i inventory.yaml wurbs -a "sudo reboot"

## Add extra features not managed by Ansible

Check the original installation instruction to install RaspAP:
https://github.com/cloudedbats/cloudedbats_wurb_2020

Note that when RaspAP is installed the WiFi connection used earlier in this instruction
is not working. The builtin WiFi module is then used by RaspAP to share it's own
WiFi network. An Ethernet cable can still be used to connect it to the local network,
and also to internet when needed.
