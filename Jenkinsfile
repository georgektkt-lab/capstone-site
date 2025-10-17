pipeline {
  agent any
  stages {
    stage('Checkout HTML files') {
      steps {
        git branch: 'main', url: 'https://github.com/georgektkt-lab/capstone-site.git'
      }
    }
    stage('Deploy with Ansible') {
      steps {
        sh '''
          cat > /opt/capstone/ansible/deploy.yml <<YAML
          - hosts: all
            become: yes
            tasks:
              - name: Ensure nginx present
                apt: { name: nginx, state: present, update_cache: yes }
              - name: Ensure nginx running
                systemd: { name: nginx, enabled: yes, state: started }
              - name: Deploy AWS HTML
                copy: { src: index-aws.html, dest: /var/www/html/index.html, owner: www-data, group: www-data, mode: "0644" }
                when: "'aws' in group_names"
              - name: Deploy Azure HTML
                copy: { src: index-azure.html, dest: /var/www/html/index.html, owner: www-data, group: www-data, mode: "0644" }
                when: "'azure' in group_names"
          YAML

          ansible-playbook -i /opt/capstone/ansible/inventory.ini /opt/capstone/ansible/deploy.yml
        '''
      }
    }
  }
}
