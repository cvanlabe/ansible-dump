# Ansible Cookbook Dump

This project is a project that captures some Ansible work I did in the past and can reuse. **Do not** consider it as a fully fledged, super optimal, exemplary Ansible project.
It's 2 years old and never updated. Today it enables me to take pieces I need them for other deployments without having to rethink them.

The workbook was built for **Ubuntu 18.04 LTS**. Ubuntu 16.04 LTS does **not** work due to an older Python 3 version. Newer versions might work, but have never been tested.

The rest of this documentation below is excerpts from previous documentation written.

# Control and run ansible from your own machine

- Install Ansible supporting Python3 `on your own workstation`:
  ```
  $ pip3 install ansible
  $ ansible --version | grep "python version"
  ```
- Add your new server DNS/IP in the `playbooks/hosts` file, in the relevant section (this is how Ansible decides which servers to connect to):

  ```
  [runon:children]
  k8smasters
  k8sworkers

  [k8smasters]
  k8s-node-1 ansible_host=10.31.160.52

  [k8sworkers]
  k8s-node-2 ansible_host=10.31.160.178
  k8s-node-3 ansible_host=10.31.160.141
  k8s-node-4 ansible_host=10.31.160.190

  [nfsservers]
  nfs-node-1 ansible_host=10.31.161.220

  [servers:children]
  runon

  [runon:vars]
  ansible_python_interpreter=/usr/bin/python3
  ```

- Run the bootstrap playbook (from your local machine):
  ```
  $ ansible-playbook bootstrap.yml
  ```
- When it finished, you should see something like
  ```
  OK! System bootstrapped... please now run any other playbooks as required!
  ```
- So we can now run the playbook for kubernetes hosts for example::
  ```
  $ ansible-playbook kubernetes.yml
  ```

You will now have a new server up and running, where all the teams userids are added, their public keys, and the main tools we typically use: docker, git, python, ... etc.

# Adding a user to the Ansible playbooks

If we have a new team member, and we need to add him/her to one or more servers:

- add the user to the file `playbooks/roles/bootstrap/vars/main.yml`
  ```
  ---
  # Users we need on the servers as sysadmins
  users:
    - name: cvanlabe
      shell: "/bin/bash"
      uid: 5001
      state: present
      groups:
        - sudo
        - docker
      ssh-keys:
        - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAAB...
  ```
  This is a YAML file, so indentation is important! Please add the user to the existing list, and increment the uid with 1 for each new user.
- Re-run the `bootstrap` Ansible Playbook. `Enter your own username`, and `leave the password field blank` (we're using SSH key authentication).

  ```
  What is the bootstrap username of your newly created server? [root]: cvanlabe
  What is the password of this user?:
  PLAY [servers] ************************************************************************
  TASK [Gathering Facts] ****************************************************************
  ...
  ```

# Add/Remove/Update a user's SSH key

Follow the same process as adding a new user. Instead of adding a new user, just add/remove/change the ssh-keys part in the file.

# Add a new node to the K8S cluster

- Launch a new instance in runon: https://...
- Give it 4vCPU, 32GB Memory - Ubuntu 18.04
- Provide it Access Groups 'default' and 'Kubernetes'
- Once it's ready, add the IP address to the `playbooks/hosts` file
- bootstrap: `ansible-playbook bootstrap.yml` -l <your new hostname>
- configure: `ansible-playbook kubernetes.yml` -l <your new hostname>
- You're done!

Enjoy!
