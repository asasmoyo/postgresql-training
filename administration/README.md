PostgreSQL Administration
=========================

# Prerequisites

1. Laptop / PC that allows you to run 2 VMs with at least 1 CPU and 1 GB RAM

1. Internet connection

1. [VMware Workstation Player](https://www.vmware.com/asean/products/workstation-player/workstation-player-evaluation.html) for Windows and Linux or [VMware Fusion Player](https://customerconnect.vmware.com/web/vmware/evalcenter?p=fusion-player-personal) for Mac

1. [Vagrant](https://www.vagrantup.com/downloads)

1. [Vagrant VMware Utility](https://www.vagrantup.com/docs/providers/vmware/vagrant-vmware-utility)

# Before D-Day

Please try to run the VMs and update apt cache before the training.

1. Open your command prompt

2. Install Vagrant VMware plugin

    ``` bash
    vagrant plugin install vagrant-vmware-desktop
    ```

3. Download Vagrant box with

    ``` bash
    vagrant box add bento/ubuntu-20.04
    ```

4. Clone this repo, then `cd` into `administration` folder

5. Bring up the VMs

    ``` bash
    vagrant up --provider vmware_desktop
    ```

6. After the VMs are up, ssh into the VMs

    Open command line then:

    ``` bash
    vagrant ssh node-1
    ```

    Open another command line then:

    ``` bash
    vagrant ssh node-2
    ```

7. Update apt cache on both VMs with:

    ``` bash
    sudo apt update
    ```
