pipeline {
    agent any
    
    environment {
        ANSIBLE_PATH = '/opt/ansible-venv/bin'
        HOST_IP = '172.17.0.1'
        FRONTEND_REPO = 'https://github.com/SolovevNik22IST1/frontend.git'
        BACKEND_REPO = 'https://github.com/SolovevNik22IST1/backend.git'
        DEVOPS_REPO = 'https://github.com/SolovevNik22IST1/DevOps.git'
    }
    
    stages {
        stage('Проверка SSH подключения') {
            steps {
                sh '''
                    ssh -i /var/jenkins_home/.ssh/id_rsa ubuntu@$HOST_IP "echo 'SSH работает'"
                '''
            }
        }
        
        stage('Подготовка Ansible проекта') {
            steps {
                sh '''
                    mkdir -p workspace/ansible
                    cat > workspace/ansible/inventory.ini << INV
[ubuntu_host]
$HOST_IP ansible_user=ubuntu ansible_ssh_private_key_file=/var/jenkins_home/.ssh/id_rsa
INV
                '''
            }
        }
        
        stage('Ansible: Проверка Docker') {
            steps {
                sh '''
                    cat > workspace/ansible/check_docker.yml << PLAYBOOK
---
- name: Проверка Docker
  hosts: ubuntu_host
  tasks:
    - name: Проверка версии Docker
      command: docker --version
      register: docker_version
    - name: Вывод версии
      debug: msg="{{ docker_version.stdout }}"
PLAYBOOK
                    $ANSIBLE_PATH/ansible-playbook workspace/ansible/check_docker.yml -i workspace/ansible/inventory.ini
                '''
            }
        }
        
        stage('Ansible: Клонирование репозиториев') {
            steps {
                sh '''
                    cat > workspace/ansible/clone_repos.yml << PLAYBOOK
---
- name: Клонирование репозиториев
  hosts: ubuntu_host
  tasks:
    - name: Создание директории
      file: path=/home/ubuntu/app state=directory
    - name: Клонирование frontend
      git: repo=https://github.com/SolovevNik22IST1/frontend.git dest=/home/ubuntu/app/frontend
    - name: Клонирование backend
      git: repo=https://github.com/SolovevNik22IST1/backend.git dest=/home/ubuntu/app/backend
    - name: Клонирование devops
      git: repo=https://github.com/SolovevNik22IST1/DevOps.git dest=/home/ubuntu/app/devops
PLAYBOOK
                    $ANSIBLE_PATH/ansible-playbook workspace/ansible/clone_repos.yml -i workspace/ansible/inventory.ini
                '''
            }
        }
        
        stage('Ansible: Запуск приложений') {
            steps {
                sh '''
                    cat > workspace/ansible/deploy_apps.yml << PLAYBOOK
---
- name: Запуск приложений
  hosts: ubuntu_host
  tasks:
    - name: Запуск docker-compose
      shell: cd /home/ubuntu/app && docker-compose up -d
PLAYBOOK
                    $ANSIBLE_PATH/ansible-playbook workspace/ansible/deploy_apps.yml -i workspace/ansible/inventory.ini
                '''
            }
        }
        
        stage('Проверка результатов') {
            steps {
                sh '''
                    ssh -i /var/jenkins_home/.ssh/id_rsa ubuntu@$HOST_IP "docker ps"
                '''
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline выполнен успешно'
        }
    }
}
