
# Interactive Playbook (Options + Await)

```yaml
---
- name: Interactive Ansible Playbook
  hosts: localhost
  gather_facts: false
  vars_prompt:
    - name: user_choice
      prompt: |
        
        Please select an option:
        A) Install web server
        B) Configure database
        C) Setup monitoring
        
        Enter your choice (A/B/C)
      private: false

  tasks:
    - name: Display selected option
      debug:
        msg: "You selected option: {{ user_choice | upper }}"

    - name: Execute option A - Install web server
      debug:
        msg: "Installing web server..."
      when: user_choice | upper == 'A'

    - name: Execute option B - Configure database
      debug:
        msg: "Configuring database..."
      when: user_choice | upper == 'B'

    - name: Execute option C - Setup monitoring
      debug:
        msg: "Setting up monitoring..."
      when: user_choice | upper == 'C'

    - name: Handle invalid option
      fail:
        msg: "Invalid option selected. Please choose A, B, or C."
      when: user_choice | upper not in ['A', 'B', 'C']

    # Later in the playbook - wait for user confirmation
    - name: Display important message to user
      pause:
        prompt: |
          
          ============================================
          IMPORTANT: Manual action required!
          
          Please complete the following steps:
          1. Log into the external system
          2. Verify the configuration
          3. Run the validation script
          
          Press ENTER when you're ready to continue...
          ============================================

    - name: Wait for user to confirm they want to proceed
      pause:
        prompt: |
          
          Are you sure you want to continue? (yes/no)
      register: user_confirmation

    - name: Verify user said yes
      fail:
        msg: "User did not confirm. Stopping playbook."
      when: user_confirmation.user_input | lower != 'yes'

    - name: Continue with remaining tasks
      debug:
        msg: "User confirmed! Continuing with playbook execution..."

    - name: Additional tasks go here
      debug:
        msg: "Executing remaining tasks..."
        
```