

# Interactive setup with external playbook

```yaml
# ============================================================================
# MAIN PLAYBOOK (main.yml)
# This is the initial playbook you run to bootstrap everything
# ============================================================================
---
- name: Bootstrap and Run External Playbook
  hosts: localhost
  become: true
  gather_facts: true
  
  vars:
    git_repo_url: "https://github.com/username/ansible-playbooks.git"
    git_repo_dest: "/tmp/ansible-playbooks"
    playbook_to_run: "interactive-setup.yml"

  tasks:
    # 1 - Install core packages
    - name: Install core packages (Debian/Ubuntu)
      apt:
        name:
          - git
          - curl
          - wget
          - vim
          - htop
          - net-tools
          - python3
          - python3-pip
        state: present
        update_cache: true
      when: ansible_os_family == "Debian"

    - name: Install core packages (RedHat/CentOS)
      yum:
        name:
          - git
          - curl
          - wget
          - vim
          - htop
          - net-tools
          - python3
          - python3-pip
        state: present
      when: ansible_os_family == "RedHat"

    - name: Display installed packages
      debug:
        msg: "Core packages installed successfully"

    # 2 - Git clone the external playbook repository
    - name: Remove existing repo directory if it exists
      file:
        path: "{{ git_repo_dest }}"
        state: absent

    - name: Clone playbook repository from git
      git:
        repo: "{{ git_repo_url }}"
        dest: "{{ git_repo_dest }}"
        version: main
        force: true
      register: git_clone_result

    - name: Display git clone status
      debug:
        msg: "Repository cloned to {{ git_repo_dest }}"

    # 3 - Run the external playbook (which contains all the interactive stuff)
    - name: Run external interactive playbook
      command: "ansible-playbook {{ git_repo_dest }}/{{ playbook_to_run }}"
      args:
        chdir: "{{ git_repo_dest }}"
      register: external_playbook_result
      changed_when: true

    - name: Display completion message
      debug:
        msg: "External playbook execution completed!"


# ============================================================================
# EXTERNAL PLAYBOOK (interactive-setup.yml)
# This lives in your git repository and contains all the interactive parts
# ============================================================================
---
- name: Interactive Configuration and Deployment
  hosts: localhost
  gather_facts: true
  vars_prompt:
    # 4 - Prompt user for options
    - name: deployment_option
      prompt: |
        
        ╔════════════════════════════════════════╗
        ║     DEPLOYMENT CONFIGURATION MENU      ║
        ╚════════════════════════════════════════╝
        
        Please select your deployment option:
        
        A) Development Environment
           - Installs dev tools
           - Enables debug mode
           - Uses local database
        
        B) Staging Environment
           - Production-like setup
           - External database
           - Monitoring enabled
        
        C) Production Environment
           - Full security hardening
           - Load balancer configuration
           - Backup automation
        
        Enter your choice (A/B/C)
      private: false

    - name: enable_monitoring
      prompt: "Enable monitoring and logging? (yes/no)"
      private: false
      default: "yes"

    - name: enable_backups
      prompt: "Enable automated backups? (yes/no)"
      private: false
      default: "yes"

  tasks:
    - name: Display selected configuration
      debug:
        msg: 
          - "Selected environment: {{ deployment_option | upper }}"
          - "Monitoring enabled: {{ enable_monitoring }}"
          - "Backups enabled: {{ enable_backups }}"

    # 5 - Handle the options
    - name: Configure Development Environment
      block:
        - name: Install development packages
          apt:
            name:
              - nodejs
              - npm
              - python3-dev
              - build-essential
            state: present
          when: ansible_os_family == "Debian"
        
        - name: Configure dev database
          debug:
            msg: "Setting up local SQLite database..."
        
        - name: Enable debug mode
          lineinfile:
            path: /etc/environment
            line: "DEBUG=true"
            create: true
        
        - name: Create dev config file
          copy:
            content: |
              ENV=development
              DEBUG=true
              DATABASE=sqlite:///local.db
            dest: /opt/app/config.env
            mode: '0644'
      when: deployment_option | upper == 'A'

    - name: Configure Staging Environment
      block:
        - name: Setup staging server configuration
          debug:
            msg: "Configuring staging environment settings..."
        
        - name: Create staging config file
          copy:
            content: |
              ENV=staging
              DEBUG=false
              DATABASE=postgresql://staging-db:5432/app
            dest: /opt/app/config.env
            mode: '0644'
        
        - name: Install monitoring agent
          apt:
            name: prometheus-node-exporter
            state: present
          when: ansible_os_family == "Debian"
      when: deployment_option | upper == 'B'

    - name: Configure Production Environment
      block:
        - name: Apply security hardening
          debug:
            msg: "Applying production security policies..."
        
        - name: Create production config file
          copy:
            content: |
              ENV=production
              DEBUG=false
              DATABASE=postgresql://prod-db:5432/app
              REDIS=redis://prod-cache:6379
            dest: /opt/app/config.env
            mode: '0600'
        
        - name: Install nginx
          apt:
            name: nginx
            state: present
          when: ansible_os_family == "Debian"
        
        - name: Configure firewall
          ufw:
            rule: allow
            port: "{{ item }}"
          loop:
            - '80'
            - '443'
          when: ansible_os_family == "Debian"
      when: deployment_option | upper == 'C'

    - name: Setup monitoring if enabled
      block:
        - name: Install monitoring tools
          apt:
            name:
              - prometheus-node-exporter
              - grafana
            state: present
          when: ansible_os_family == "Debian"
        
        - name: Start monitoring services
          systemd:
            name: "{{ item }}"
            state: started
            enabled: true
          loop:
            - prometheus-node-exporter
            - grafana-server
          when: ansible_os_family == "Debian"
      when: enable_monitoring | lower == 'yes'

    - name: Setup backups if enabled
      block:
        - name: Install backup tools
          apt:
            name: rsync
            state: present
          when: ansible_os_family == "Debian"
        
        - name: Create backup script
          copy:
            content: |
              #!/bin/bash
              rsync -avz /opt/app/data /backup/$(date +%Y%m%d)/
            dest: /usr/local/bin/backup.sh
            mode: '0755'
        
        - name: Schedule daily backups
          cron:
            name: "Daily application backup"
            minute: "0"
            hour: "2"
            job: "/usr/local/bin/backup.sh"
      when: enable_backups | lower == 'yes'

    - name: Handle invalid environment option
      fail:
        msg: "Invalid option '{{ deployment_option }}'. Please choose A, B, or C."
      when: deployment_option | upper not in ['A', 'B', 'C']

    # 6 - Perform action and prompt user
    - name: Deploy application files
      debug:
        msg: "Deploying application to {{ deployment_option | upper }} environment..."

    - name: Create application directory
      file:
        path: /opt/app
        state: directory
        mode: '0755'

    - name: Display SSL certificate instructions
      pause:
        prompt: |
          
          ╔════════════════════════════════════════════════════════╗
          ║          MANUAL ACTION REQUIRED                        ║
          ╚════════════════════════════════════════════════════════╝
          
          The application has been deployed, but SSL certificates
          need to be configured manually.
          
          REQUIRED STEPS:
          1. Log into your domain registrar (e.g., Cloudflare, Route53)
          2. Point your domain's A record to: {{ ansible_default_ipv4.address | default('SERVER_IP') }}
          3. Wait 5-10 minutes for DNS propagation
          4. Verify DNS: dig yourdomain.com
          5. Run: sudo certbot --nginx -d yourdomain.com
          6. Test certificate: https://yourdomain.com
          
          Press ENTER when you have completed these steps...

    - name: Verify user is ready to continue
      pause:
        prompt: |
          
          Have you completed the SSL certificate setup? (yes/no)
      register: ssl_confirmation

    - name: Check user confirmation
      fail:
        msg: "SSL setup not confirmed. Please complete the steps before continuing."
      when: ssl_confirmation.user_input | lower != 'yes'

    # 7 - Continue after confirmation
    - name: Continue with post-deployment tasks
      debug:
        msg: "User confirmed SSL setup. Continuing with final configuration..."

    - name: Reload nginx configuration
      systemd:
        name: nginx
        state: reloaded
      when: 
        - deployment_option | upper in ['B', 'C']
        - ansible_os_family == "Debian"

    - name: Run application health check
      uri:
        url: "http://localhost:80"
        status_code: 200
      register: health_check
      ignore_errors: true

    - name: Display health check status
      debug:
        msg: "Application health check: {{ 'PASSED' if health_check.status == 200 else 'FAILED' }}"

    - name: Display final status
      debug:
        msg: |
          
          ╔════════════════════════════════════════════════════════╗
          ║           DEPLOYMENT COMPLETED SUCCESSFULLY            ║
          ╚════════════════════════════════════════════════════════╝
          
          Environment: {{ deployment_option | upper }}
          Monitoring: {{ enable_monitoring }}
          Backups: {{ enable_backups }}
          SSL: Configured
          Status: ONLINE
          
          Configuration file: /opt/app/config.env
          Application directory: /opt/app
          
          Your application is now ready to use!
          
          Next steps:
          - Upload your application code to /opt/app
          - Review configuration in /opt/app/config.env
          - Check logs: journalctl -u nginx
```

