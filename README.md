# Ansible Playbook to Deploy a Service Fabric Cluster on AWS
This project contains the Ansible devops playbooks, templates and files
to spin up a Service Fabric cluster of any specified size on AWS. The
consumer of the playbook can specify the number of machines, the sizes
of the machines, and the AWS deployment region. By default, the playbook
uses the following:

```yml
# extracted from ./site.yml

ami_id: ami-a9a8e4c9            # EBS Optimized Ubuntu 16.04
key_name: demo-ssh-key          # name of the public key to create (uses ~/.ssh/id_rsa.pub)
aws_region: us-west-1           # the deployment region
aws_tag: service-fabric-demo    # tag used to count instances
instance_type: m4.large         # using 2 CPU, 8 GB, 450 Mbps EBS backed
instance_count: 2               # number of machines
```

## Setup for AWS and Ansible
- [Install Python 2.7 and Pip](https://www.python.org/downloads/)
- [Install Ansible](http://docs.ansible.com/ansible/intro_installation.html)
- [Install Boto](https://github.com/boto/boto)
- [Setup your AWS credentials for Boto](https://github.com/boto/boto#getting-started-with-boto)
- Clone this repo and change to the root directory of the repo
- Test boto connectivity with `./ec2.py --list`

## Deploy a Cluster
- From the root directory of this repo
- `ansible-playbook site.yml -i ec2.py`

## Using the cluster
- Log on to the cluster `ssh ubuntu@{cluster ip}`
- Setup an ssh tunnel to view the dashboard locally `ssh -N -p 22 -C ubuntu@{cluster ip} -L 19080:localhost:19080`
    - After starting the tunnel, open your local browser to http://localhost:19080
- To deploy to the cluster
    - you can setup additional ssh tunnels and work locally
    - you can open external ports to expose the cluster publicly (by default, the only public port open is 22)
- Example
    - `ssh ubuntu@{cluster ip}`
    - `yo azuresf`
        - name the app `testapp`
        - select `Reliable Actor Service - Java`
        - name the actor `testactor`
    - `cd testapp`
    - `gradle`
    - `azuresfcli config mode arm`
    - `azuresfcli servicefabric cluster connect http://localhost:19080`
    - `./install.sh`
    - Look to your dashboard and you should see your application running in the cluster.
    - $$$$

## Teardown a Cluster
- From the root directory of this repo
- `ansible-playbook terminate.yml -i ec2.py`

## Other Details
Here are some other details to help out anyone not familiar with Ansible.

### Project Structure
```
├── LICENSE
├── README.md
├── ec2.ini
├── ec2.py
├── roles
│   ├── common
│   │   └── tasks
│   │       └── main.yml
│   ├── provision-ec2
│   │   └── tasks
│   │       └── main.yml
│   └── service-fabric
│       ├── tasks
│       │   └── main.yml
│       └── templates
│           └── manifest.xml.j2
├── site.yml
└── terminate.yml
```
- **site.yml** is the start of all deployment activity. It includes three
  roles, provision-ec2, common and service-fabric.
- **Roles**
    - **provision-ec2:** handles all of the AWS related provisioning
    - **common:** handles bootstrapping the initial Ubuntu image (adds Python 2.7)
    - **service-fabric:** installs and builds the SF cluster
        - **tasks** contains the tasks to be executed during deployment
        - **templates** contains jinja2 template to be deployed to the cluster
- **terminate.yml** destroys clusters
- **ec2.ini** contains boto / ec2 configuration defaults
- **ec2.py** is an inventory script for pulling remote machine inventory from
  AWS
