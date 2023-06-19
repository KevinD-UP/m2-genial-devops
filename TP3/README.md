## 3-1 Document your inventory and base commands
```yml
all:
  vars:
    ansible_user: centos
    ansible_ssh_private_key_file: ../../TP3/id_rsa
  children:
    prod:
      hosts: kevindang.takima.cloud
```
- The all keyword represents the top-level group that includes all hosts in the inventory.
- Under vars, two variables are defined:
    - ansible_user: centos sets the SSH user to "centos" for all hosts.
    - ansible_ssh_private_key_file: ../../TP3/id_rsa specifies the path to the private SSH key file to be used for authentication on all hosts. It points to ../../TP3/id_rsa relative to the location of the inventory file.
- The children section defines child groups.
- Under prod, there is a single host entry:
    - hosts: kevindang.takima.cloud assigns the host "kevindang.takima.cloud" to the "prod" group.

This inventory file sets up the variables and host configuration for your Ansible playbook. It assumes the presence of a private SSH key file at the specified location, and it configures the SSH user as "centos" for all hosts.

## 3-2 Document your playbook

```yml
- name: Clean packages
  command:
    cmd: yum clean -y packages

- name: Install device-mapper-persistent-data
  yum:
    name: device-mapper-persistent-data
    state: latest

- name: Install lvm2
  yum:
    name: lvm2
    state: latest

- name: add repo docker
  command:
    cmd: sudo yum-config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo

- name: Install Docker
  yum:
    name: docker-ce
    state: present

- name: install python3
  yum:
    name: python3

- name: Pip install
  pip:
    name: docker
    executable: pip3

- name: Make sure Docker is running
  service: name=docker state=started
  tags: docker
```

Task: Clean packages

Description: Cleans the yum package cache.
Command: yum clean -y 

Task: Install device-mapper-persistent-data

Description: Installs the device-mapper-persistent-data package using yum.
Package: device-mapper-persistent-data
State: latest

Task: Install lvm2

Description: Installs the lvm2 package using yum.
Package: lvm2
State: latest

Task: Add repo docker

Description: Adds the Docker repository to the yum configuration.
Command: sudo yum-config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
Task: Install Docker

Description: Installs Docker CE using yum.
Package: docker-ce
State: present

Task: Install python3

Description: Installs the Python 3 package using yum.
Package: python3

Task: Pip install

Description: Installs the Docker Python package using pip3.
Package: docker
Executable: pip3

Task: Make sure Docker is running

Description: Ensures that the Docker service is running.
Service: docker
State: started
Tags: docker

These tasks perform various actions such as cleaning the package cache, installing necessary packages (device-mapper-persistent-data, lvm2, docker-ce, python3, and docker Python package), and ensuring that the Docker service is running.

## 3-3 Document your docker_container tasks configuration.

```yml
- hosts: all
  gather_facts: false
  become: yes
  tasks:
    - name: Import Docker role
      import_role:
        name: docker
    - name: Import network role
      import_role:
        name: network        
    - name: Import database role
      import_role:
        name: database
    - name: Import backend role
      import_role:
        name: backend
    - name: Import httpd role
      import_role:
        name: httpd
    - name: Import frontend role
      import_role:
        name: frontend
```

Let's go through the playbook step by step:

- hosts: all specifies that the playbook will be executed on all hosts defined in the inventory file.
- gather_facts: false disables the gathering of facts about the target hosts. Facts include information such as IP addresses, operating system details, etc.
- become: yes enables privilege escalation, allowing the tasks to be run with administrative (root) privileges. This is typically required for tasks that involve system configuration or installation.
- tasks is a list of tasks to be executed in sequence.
Each task is defined using the name parameter to provide a descriptive name for the task.
- The import_role module is used to import and execute roles defined in separate directories.
- For each task, the import_role module is invoked with the name parameter specifying the name of the role to import.

Each role is using docker_container to pull and run an image, for example for the backend :

```yml
# tasks file for roles/backend
- name: Run backend container
  docker_container:
    name: backend
    image: kevindang01/backend:latest
    networks:
      - name: my_network
        aliases: ['backend']
```

For the rest :

database 

```yml
---
# tasks file for roles/database
- name: Run Database container
  docker_container:
    name: database
    image: kevindang01/database:latest
    networks:
      - name: my_network
        aliases: ['database']
    # Add other necessary parameters (e.g., environment variables, volumes, ports, etc.)
    env:
      POSTGRES_DB: db
      POSTGRES_USER: usr
      POSTGRES_PASSWORD: pwd 
    volumes:
      - my_volume:/var/liv/postgresql/data
```

docker

```yml
# tasks file for roles/docker
- name: Clean packages
  command:
    cmd: yum clean -y packages

- name: Install device-mapper-persistent-data
  yum:
    name: device-mapper-persistent-data
    state: latest

- name: Install lvm2
  yum:
    name: lvm2
    state: latest

- name: add repo docker
  command:
    cmd: sudo yum-config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo

- name: Install Docker
  yum:
    name: docker-ce
    state: present

- name: install python3
  yum:
    name: python3

- name: Pip install
  pip:
    name: docker
    executable: pip3

- name: Make sure Docker is running
  service: name=docker state=started
  tags: docker

```

frontend
```yml
# tasks file for roles/frontend
- name: Run frontend container
  docker_container:
    name: devops-front
    image: kevindang01/devops-front:latest
    networks:
      - name: my_network
        aliases: ['devops-front']
```

httpd
```yml
# tasks file for roles/http-server
- name: Run httpd container
  docker_container:
    name: httpd
    image: kevindang01/httpd:latest
    networks:
      - name: my_network
        aliases: ['httpd']
    # Add other necessary parameters (e.g., environment variables, volumes, ports, etc.)
    ports:
      - 80:80
```

network
```yml
# tasks file for roles/network
- name: Create Docker network
  docker_network:
    name: my_network
```

volume
```yml
# tasks file for roles/volume
- name: Create Docker volume
  docker_volume:
    name: my_volume
``` 

To run the playbook (from the ansible directory):

```bash
ansible-playbook -i inventories/setup.yml playbook.yml
```