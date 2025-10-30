
# Pretty printing

```
- name: qrencode output
  hosts: localhost
  tasks:
  - name: capture qrencode output
    register: results
    ansible.builtin.command: qrencode 'hunter2' -t utf8
  - name: display output
    ansible.builtin.debug: # as is this
      var: results.stdout_lines # this is key
```