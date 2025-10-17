pipeline {
  agent any

  options { timestamps() }

  environment {
    PATH = "/usr/bin:/usr/sbin:${env.PATH}"
    AWS_IP = '54.157.10.77'
    AZURE_IP = '172.171.2.98'
    AWS_KEY = '/var/lib/jenkins/.ssh/aws_capstone'
    AZURE_KEY = '/var/lib/jenkins/.ssh/azure_capstone'
    INV = "${env.WORKSPACE}/inventory.ini"
    PLAY = "${env.WORKSPACE}/deploy.yml"
  }

  stages {
    stage('Verify checkout & tools') {
      steps {
        sh '''
          echo "WORKSPACE=$WORKSPACE"
          ls -la "$WORKSPACE"

          echo "Checking Ansible:"
          which ansible-playbook || true
          ansible-playbook --version || true

          echo "Check Jenkins SSH keys exist:"
          ls -l $AWS_KEY $AZURE_KEY
        '''
      }
    }

    stage('Create inventory & playbook (in workspace)') {
      steps {
        sh '''
          # Write inventory here (no sudo)
          cat > "$INV" <<EOF
[aws]
aws-app ansible_host=${AWS_IP} ansible_user=ubuntu ansible_ssh_private_key_file=${AWS_KEY}

[azure]
azure-vm ansible_host=${AZURE_IP} ansible_user=azureuser ansible_ssh_private_key_file=${AZURE_KEY}
EOF

          echo "Inventory written to $INV:"
          sed -n '1,120p' "$INV"

          # Write playbook here (no sudo)
          cat > "$PLAY" <<'EOF'
- hosts: all
  become: yes
  vars:
    ws: "{{ ws }}"
  tasks:
    - name: Ensure nginx present
    # 'update_cache' on the first run, then fast next runs
      apt:
        name: nginx
        state: present
        update_cache: yes

    - name: Ensure nginx running
      systemd:
        name: nginx
        enabled: yes
        state: started

    - name: Deploy AWS HTML
      copy:
        src: "{{ ws }}/index-aws.html"
        dest: /var/www/html/index.html
        owner: www-data
        group: www-data
        mode: "0644"
      when: "'aws' in group_names"

    - name: Deploy Azure HTML
      copy:
        src: "{{ ws }}/index-azure.html"
        dest: /var/www/html/index.html
        owner: www-data
        group: www-data
        mode: "0644"
      when: "'azure' in group_names"
EOF

          echo "Playbook written to $PLAY:"
          sed -n '1,200p' "$PLAY"
        '''
      }
    }

    stage('Deploy with Ansible') {
      steps {
        sh '''
          ansible-playbook -i "$INV" "$PLAY" --extra-vars "ws=$WORKSPACE"
        '''
      }
    }
  }

  post {
    success {
      echo '✅ Deployed!'
      echo 'Check:'
      echo '  http://54.157.10.77  (AWS)'
      echo '  http://172.171.2.98  (Azure)'
    }
    failure {
      echo '❌ Build failed — see Console Output.'
    }
  }
}

