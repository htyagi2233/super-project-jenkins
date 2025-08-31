# ✅ LAB OVERVIEW
complete lab setup to install Docker, Git, and Maven on a test server using Jenkins Pipeline + Ansible.

**Component	Description**
- Server 1	Jenkins + Ansible control node
- Server 2	Target/Test node (where tools are installed)
- Jenkins Job	Pipeline triggering Ansible playbook


## 🔧 1. PREREQUISITES
### 💻 Two Ubuntu Servers (e.g., Ubuntu 20.04 or 22.04)

- Server-1: Jenkins + Ansible (192.168.192.130)
- Server-2: Target/test machine (192.168.192.137)

## 🚀 2. SETUP SERVER-1 (JENKINS + ANSIBLE)
###  A. Install Jenkins
```
sudo apt update
sudo apt install -y openjdk-11-jdk
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update
sudo apt install -y jenkins
sudo systemctl enable jenkins
sudo systemctl start jenkins
```

### 🔹 B. Install Ansible
```
sudo apt install -y ansible
```

### 🔹 C. Generate SSH Key for Ansible
```
ssh-keygen -t rsa -b 2048 -f ~/.ssh/jenkins_ansible -N ""
```


This creates:

- Private key: /home/your-user/.ssh/jenkins_ansible
- Public key: /home/your-user/.ssh/jenkins_ansible.pub

You will use this key for Ansible/Jenkins to access the test server.

## 🧩 3. CONFIGURE SERVER-2 (TEST SERVER)
### 🔹 A. Add Public Key to Root's Authorized Keys

On Server-2 (192.168.192.137):
```
mkdir -p /root/.ssh
echo "<CONTENTS_OF_jenkins_ansible.pub>" >> /root/.ssh/authorized_keys
chmod 600 /root/.ssh/authorized_keys
chmod 700 /root/.ssh
```


Make sure PermitRootLogin yes is enabled in /etc/ssh/sshd_config, and restart SSH if needed:
```
sudo systemctl restart ssh
```

### 🔹 B. Test SSH from Server-1

From Server-1:
```
ssh -i ~/.ssh/jenkins_ansible root@192.168.192.137
```

If this works, you're good to go.

## 🔐 4. ADD SSH KEY TO JENKINS CREDENTIALS
In Jenkins UI:

- Go to Manage Jenkins → Credentials → (Global) → Add Credentials
- Type: SSH Username with private key
- ID: jenkins-ansible-key
- Username: root
- Private Key: Paste contents of ~/.ssh/jenkins_ansible

## 📁 5. GITHUB PROJECT STRUCTURE
```
super-project-jenkins/
│
├── TestServerAnsible/
│   ├── inventory
│   └── playbook.yaml

```

### 🔹 A. inventory (INI format)
```
[all:vars]
ansible_ssh_common_args='-o StrictHostKeyChecking=no'

[test]
test ansible_ssh_host=192.168.192.137 ansible_ssh_port=22 ansible_user=root
```

### 🔹 B. playbook.yaml
```
---
- name: Install Docker, Git, and Maven on Ubuntu
  hosts: test
  become: true
  tasks:
    - name: Update APT cache
      apt:
        update_cache: yes

    - name: Install Docker
      apt:
        name: docker.io
        state: present

    - name: Install Git
      apt:
        name: git
        state: present

    - name: Install Maven
      apt:
        name: maven
        state: present

```

## 🧪 6. CREATE JENKINS PIPELINE JOB

In Jenkins UI:
- Create a Pipeline job.
- Add pipeline script (or use a Jenkinsfile in your repo):
```
pipeline {
    agent any
    environment {
        privatekey = credentials('jenkins-ansible-key')
    }
    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/htyagi2233/super-project-jenkins.git'
            }
        }
        stage('Install Tools on Test Server') {
            steps {
                sh '''
                cd TestServerAnsible
                ansible-playbook -i inventory --private-key ${privatekey} playbook.yaml
                '''
            }
        }
    }
}

```

## ✅ 7. RUN AND VERIFY

- Run the Jenkins job.
- It should SSH into the test server and install Docker, Git, and Maven.
- You can verify by running the following on the test server:

```
docker --version
git --version
mvn --version
```

## 🛠️ 8. OPTIONAL: PRE-TEST WITH ANSIBLE PING

You can add this as a test stage before installation:
```
- name: Ping test host
  hosts: test
  tasks:
    - name: Ping
      ping:
```

Or as a separate Jenkins stage.

## 🔁 9. TROUBLESHOOTING
- Issue	      Solution
- SSH         unreachable	Check key, host IP, firewall, SSH config
- Permission  denied	Use correct user and keys
- Ansible     error	Use ansible-playbook -vvv for debug
- Jenkins     can't find ansible	Add Ansible to Jenkins path or use full path /usr/bin/ansible-playbook
