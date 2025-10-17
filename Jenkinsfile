pipeline {
  agent any

  options {
    timestamps()
  }

  environment {
    INV = '/opt/capstone/ansible/inventory.ini'
    DEPLOY_PLAY = '/opt/capstone/ansible/deploy.yml'
    PATH = "/usr/bin:/usr/sbin:${env.PATH}"
  }

  stages {
    stage('Verify checkout & tools') {
      steps {
        sh '''
          echo "WORKSPACE=$WORKSPACE"
          ls -la "$WORKSPACE"

          echo "Ansible version:"
          which ansible-playbook || true
          ansible-playbook --version || true

          echo "Inventory preview:"
          test -f "$INV" && sed -n '1,120p' "$INV" || (echo "Inventory not found at $INV" && exit 1)
        '''
      }
    }

    stage('Create playbook') {
      steps {
        sh '''
          sudo mkdir -p /opt/capstone/ansible
          sudo chown jenkins:jenkins /opt/capstone/ansible

          cat > "$DEPLOY_PLAY" <<'EOF'
          - hosts: all
            become: yes
            vars:
              ws: "{{ ws }}"
            tasks:
              - name: Ensure nginx present
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

          echo "Wrote playbook to $DEPLOY_PLAY"
        '''
      }
    }

    stage('Deploy with Ansible') {
      steps {
        sh '''
          ansible-playbook -i "$INV" "$DEPLOY_PLAY" --extra-vars "ws=$WORKSPACE"
        '''
      }
    }
  }

  post {
    success {
      echo '✅ Deployment complete.'
      echo 'Test:'
      echo '  - http://54.157.10.77  (AWS)'
      echo '  - http://172.171.2.98  (Azure)'
    }
    failure {
      echo '❌ Build failed. Check console for details.'
    }
  }
}

