# 🎯 Objective:
- ✅ Jenkins pipeline →
- ✅ Ansible playbook trigger →
- ✅  SonarQube install via Ansible on Remote server (Sonar)


## 🧱 Pre Requirements:
- Jenkins server (Linux-based preferred)
- Ansible installed on Jenkins server
- GitHub repository → Ansible playbook (for SonarQube)
- SSH Agent Plugin install on jenkins
- SSH access from Jenkins to target (SonarQube) server
- Jenkins → SSH credentials configured
- Jenkins job → Pipeline (Scripted or Declarative)


# Lab Setup: Jenkins + Ansible + Passwordless SSH

## Assumptions:
	- Jenkins and Ansible are installed on 192.168.192.130.
	- Remote server for SonarQube is 192.168.192.135.
	- Jenkins user’s home directory is /var/lib/jenkins.
	- You want to connect to the remote server as root.
	- Jenkins user has sudo rights (optional if not connecting as root).

## Step 1: Generate SSH Key for Jenkins User
Generate an SSH key for Jenkins user so it can connect to the remote server without a password.
```
sudo -u jenkins ssh-keygen -t rsa -b 4096 -N "" -f /var/lib/jenkins/.ssh/id_rsa
```

	• -N "" means no passphrase.
	• Private key saved as /var/lib/jenkins/.ssh/id_rsa
	• Public key saved as /var/lib/jenkins/.ssh/id_rsa.pub

## Step 2: Copy Jenkins Public Key to Remote Server’s root Authorized Keys
Copy Jenkins public key into the remote server’s root user authorized keys, so Jenkins can login passwordlessly:
On Jenkins server:
```
sudo cat /var/lib/jenkins/.ssh/id_rsa.pub
```

Copy the output, then on the remote server (192.168.192.135):
```
ssh root@192.168.192.135
mkdir -p ~/.ssh
chmod 700 ~/.ssh
echo "paste the Jenkins public key here" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

## Step 3: Add Remote Server SSH Key to Jenkins Known Hosts
To avoid SSH host verification errors:
```
sudo -u jenkins ssh-keyscan -H 192.168.192.135 >> /var/lib/jenkins/.ssh/known_hosts
```

## Step 4: Fix SSH Permissions for Jenkins User
Set correct permissions on Jenkins user’s .ssh folder and keys:
```
sudo chown -R jenkins:jenkins /var/lib/jenkins/.ssh
sudo chmod 700 /var/lib/jenkins/.ssh
sudo chmod 600 /var/lib/jenkins/.ssh/id_rsa
sudo chmod 644 /var/lib/jenkins/.ssh/id_rsa.pub
sudo chmod 644 /var/lib/jenkins/.ssh/known_hosts
```

## Jenkins Pipeline
```
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git url: 'checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/htyagi2233/super-project-jenkins.git']])'
            }
        }
        stage('Run Ansible') {
            steps {
                sshagent(['jenkins-ansible-key']) {
                    sh '''
                        cd sonarqube-ansible
                        ansible-playbook -i inventory install-sonarqube.yml
                    '''
                }
            }
        }
    }
}

```





## 📦 1. GitHub - Ansible playbook:
```
sonarqube-ansible/
├── install-sonarqube.yml
├── inventory
└── roles/
```


## 📁 2. install-sonarqube.yml (Ansible Playbook)
```
---
- name: Install SonarQube on Ubuntu
  hosts: sonarqube
  become: true

  vars:
    sonarqube_version: "10.1.0.73491"
    sonarqube_user: sonar
    sonarqube_group: sonar
    install_dir: /opt/sonarqube

  tasks:
    - name: Install dependencies
      apt:
        name:
          - openjdk-17-jdk
          - unzip
          - wget
        update_cache: yes

    - name: Create sonar user
      user:
        name: "{{ sonarqube_user }}"
        system: yes
        shell: /bin/bash
        create_home: yes

    - name: Download SonarQube
      get_url:
        url: "https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-{{ sonarqube_version }}.zip"
        dest: "/tmp/sonarqube.zip"

    - name: Extract SonarQube
      unarchive:
        src: /tmp/sonarqube.zip
        dest: /opt/
        remote_src: yes

    - name: Rename folder
      command: mv /opt/sonarqube-{{ sonarqube_version }} {{ install_dir }}
      args:
        creates: "{{ install_dir }}"

    - name: Change ownership
      file:
        path: "{{ install_dir }}"
        owner: "{{ sonarqube_user }}"
        group: "{{ sonarqube_group }}"
        recurse: yes

    - name: Setup systemd service
      copy:
        dest: /etc/systemd/system/sonarqube.service
        content: |
          [Unit]
          Description=SonarQube service
          After=network.target

          [Service]
          Type=forking
          ExecStart={{ install_dir }}/bin/linux-x86-64/sonar.sh start
          ExecStop={{ install_dir }}/bin/linux-x86-64/sonar.sh stop
          User={{ sonarqube_user }}
          Group={{ sonarqube_group }}
          Restart=always
          LimitNOFILE=65536

          [Install]
          WantedBy=multi-user.target

    - name: Reload systemd and enable SonarQube
      systemd:
        daemon_reload: yes
        name: sonarqube
        enabled: yes
        state: started

```

## 📁 3. inventory
```
[sonar]
192.168.192.135 ansible_user=root ansible_ssh_private_key_file=~/.ssh/id_rsa ansible_python_interpreter=/usr/bin/python3.12

```
- 🔸 192.168.1.100 → SonarQube server IP
- 🔸 ansible_user → SSH user
- 🔸 ansible_ssh_private_key_file → Jenkins/Ansible SSH key


