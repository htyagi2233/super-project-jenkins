# 🎯 Objective:
- ✅ Jenkins pipeline →
- ✅ Ansible playbook trigger →
- ✅  SonarQube install via Ansible on Remote server (Sonar)


## 🧱 Pre Requirements:
- Jenkins server (Linux-based preferred)
- Ansible installed on Jenkins server
- GitHub repository → Ansible playbook (for SonarQube)
- SSH access from Jenkins to target (SonarQube) server
- Jenkins → SSH credentials configured
- Jenkins job → Pipeline (Scripted or Declarative)


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
192.168.192.135 ansible_user=root ansible_ssh_private_key_file=~/.ssh/id_rsa
```
- 🔸 192.168.1.100 → SonarQube server IP
- 🔸 ansible_user → SSH user
- 🔸 ansible_ssh_private_key_file → Jenkins/Ansible SSH key


