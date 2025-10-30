

# Interactive setup with external playbook

```yaml
# ============================================================================
# MAIN PLAYBOOK (main.yml)
# Run this first to bootstrap everything
# ============================================================================
---
- name: Bootstrap Setup
  hosts: localhost
  become: true
  
  vars:
    git_repo_url: "https://github.com/username/my-scripts.git"
    git_repo_dest: "/tmp/my-scripts"

  tasks:
    - name: Install core packages
      apt:
        name:
          - git
          - curl
          - vim
        state: present
        update_cache: true

    - name: Clone scripts repository
      git:
        repo: "{{ git_repo_url }}"
        dest: "{{ git_repo_dest }}"
        version: main
        force: true

    - name: Run the interactive playbook
      command: "ansible-playbook {{ git_repo_dest }}/setup.yml"
      args:
        chdir: "{{ git_repo_dest }}"


# ============================================================================
# INTERACTIVE PLAYBOOK (setup.yml)
# This lives in your git repo with your scripts
# ============================================================================
---
- name: Interactive Setup
  hosts: localhost
  gather_facts: false
  vars_prompt:
    - name: version_choice
      prompt: |
        
        ╔════════════════════════════════════════════════════════╗
        ║           VERSION SELECTION MENU                       ║
        ╚════════════════════════════════════════════════════════╝
        
        Please select which version to install:
        
        1) Version 1 (Stable)
           └─ Production-ready, fully tested
        
        2) Version 2 (Beta)
           └─ Latest features, mostly stable
        
        3) Version 3 (Experimental)
           └─ Cutting edge, may have bugs
        
        ════════════════════════════════════════════════════════
        Enter your choice (1/2/3)
      private: false

  tasks:
    - name: Display selected version
      debug:
        msg: "Installing Version {{ version_choice }}"

    # Handle version 1
    - name: Install Version 1
      script: scripts/deploy/install-v1.sh
      args:
        executable: /bin/bash
      register: output
      when: version_choice == '1'

    # Handle version 2
    - name: Install Version 2
      script: scripts/deploy/install-v2.sh
      args:
        executable: /bin/bash
      register: output
      when: version_choice == '2'

    # Handle version 3
    - name: Install Version 3
      script: scripts/deploy/install-v3.sh
      args:
        executable: /bin/bash
      register: output
      when: version_choice == '3'

    - name: Show installation output
      debug:
        var: output.stdout_lines

    # Now pause and wait for user action
    - name: Wait for user to complete manual step
      pause:
        prompt: |
          
          ╔════════════════════════════════════════════════════════╗
          ║                                                        ║
          ║          🔧  MANUAL ACTION REQUIRED  🔧                ║
          ║                                                        ║
          ╚════════════════════════════════════════════════════════╝
          
          Installation has completed successfully!
          
          Before continuing, please complete these steps:
          
          ┌────────────────────────────────────────────────────────┐
          │  STEP 1: Open your web browser                         │
          │          Navigate to: http://localhost:8080            │
          │                                                         │
          │  STEP 2: Complete the setup wizard                     │
          │          Follow the on-screen instructions             │
          │                                                         │
          │  STEP 3: Copy your API key                             │
          │          Save it somewhere safe!                       │
          └────────────────────────────────────────────────────────┘
          
          ════════════════════════════════════════════════════════
          Press ENTER when you have completed all steps...
          ════════════════════════════════════════════════════════

    - name: Ask user to confirm
      pause:
        prompt: |
          
          ╔════════════════════════════════════════════════════════╗
          ║              CONFIRMATION REQUIRED                     ║
          ╚════════════════════════════════════════════════════════╝
          
          Have you completed all the setup steps above?
          
          Type 'yes' to continue or 'no' to exit
      register: confirmation

    - name: Check confirmation
      fail:
        msg: "Setup not confirmed. Exiting."
      when: confirmation.user_input | lower != 'yes'

    # Continue after confirmation
    - name: Run post-install script
      script: scripts/deploy/finalize.sh
      args:
        executable: /bin/bash

    - name: All done!
      debug:
        msg: |
          
          ╔════════════════════════════════════════════════════════╗
          ║                                                        ║
          ║          ✅  SETUP COMPLETED SUCCESSFULLY  ✅          ║
          ║                                                        ║
          ╚════════════════════════════════════════════════════════╝
          
          Your installation is complete and ready to use!
          
          Version installed: {{ version_choice }}
          Status: Online
          
          Thank you for using this installer!
```

