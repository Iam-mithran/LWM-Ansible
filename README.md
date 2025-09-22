# 🚀 Part 1 – Ansible Demo

This README serves as a **complete hands-on guide** for Ansible learning. It includes YAML manifests and command-line usage to help you practice Ansible related topics.

## 📚 What You’ll Learn

- ✅ Installing Ansible
- ✅ How to use Adhoc Commands
- ✅ Writing your First Playbook
- ✅ Ansible loops
- ✅ Run or skip specific tasks with tags

---

## 📚 Ansible Ad-Hoc Commands

### ⚡ 1️⃣ Basic Ping Test

Check connectivity to all servers listed in the inventory file slaves.txt:

```bash
ansible all -i slaves.txt -m ping
```

- all – run on all hosts in the inventory.
- -i – specify the inventory file.
- -m ping – use the ping module to test connection.

---

### 🖥️ 2️⃣ Run Simple Commands

Use the -a (action/command) option to execute shell commands:

```bash
ansible all -i slaves.txt -a "uname -a"
ansible all -i slaves.txt -a "uptime"
```

---

### 🏷️ 3️⃣ Grouping Hosts

You can group servers inside the inventory file:

**slaves.txt**

```ini
[greens]
1
2

[app]
3
4

[db]
5
6
```

Run a command on a specific group:

```bash
ansible greens -i slaves.txt -m ping
```

---

### 🌐 4️⃣ Install & Manage Apache

Install Apache (`httpd`) on the *greens* group:

```bash
ansible greens -i slaves.txt -m yum -a "name=httpd state=present"
```

* `state=present` → install package
* `state=absent`  → uninstall package

Start Apache service:

```bash
ansible greens -i slaves.txt -m service -a "name=httpd state=started"
```

Other service states:

* `stopped` → stop service
* `restarted` → restart service

---

### 🔑 5️⃣ Privilege Escalation (`-b`)

Add `-b` to become root (sudo):

```bash
ansible greens -i slaves.txt -m yum -a "name=httpd state=present" -b
ansible greens -i slaves.txt -m service -a "name=httpd state=started" -b
```

---

### 👤 6️⃣ User & File Management

Create a user:

```bash
ansible all -i slaves.txt -m user -a "name=greens password=greens123" -b
```

Copy a file to all nodes:

```bash
ansible all -i slaves.txt -m copy -a "src=./slaves.txt dest=/tmp/slaves.txt" -b
```

---

### 🔎 Quick Reference Table

| Task                 | Command Example                                                                   |
| -------------------- | --------------------------------------------------------------------------------- |
| Ping all hosts       | `ansible all -i slaves.txt -m ping`                                               |
| Check uptime         | `ansible all -i slaves.txt -a "uptime"`                                           |
| Install Apache       | `ansible greens -i slaves.txt -m yum -a "name=httpd state=present" -b`            |
| Start Apache service | `ansible greens -i slaves.txt -m service -a "name=httpd state=started" -b`        |
| Create user          | `ansible all -i slaves.txt -m user -a "name=greens password=greens123" -b`        |
| Copy file to /tmp    | `ansible all -i slaves.txt -m copy -a "src=./slaves.txt dest=/tmp/slaves.txt" -b` |

---

## 📚 Playbook First Steps

### 1️⃣ Play: Install & start Apache, copy index.html

```yaml
- hosts: qa
  become: yes
  tasks:
    - name: Install Apache Webserver
      yum:
        name: httpd
        state: present

    - name: Start Apache Webserver
      service:
        name: httpd
        state: started

    - name: Transfer html files
      copy:
        src: index.html
        dest: /var/www/html
        mode: '0777'
```

### 2️⃣ Play: Install multiple softwares with loop & Jinja2

```yaml
- hosts: all
  become: yes
  tasks:
    - name: Install multiple softwares
      yum:
        name: "{{ item }}"    # Jinja2 templating
        state: present
      loop:
        - php
        - mysql
        - unzip
        - httpz   # <-- Deliberate typo for demo (will fail)
```

### 3️⃣ Play: Using tags

```yaml
- hosts: dev
  become: yes
  tasks:
    - name: 1st Task
      debug:
        msg: Running 1st Task
      tags: green

    - name: 2nd Task
      debug:
        msg: Running 2nd Task
      tags: black

    - name: 3rd Task
      debug:
        msg: Running 3rd Task
      tags:
        - green
        - black

    - name: Notification Task
      debug:
        msg: Send notification
      tags: notify
```

---

### 📺 Watch the Video

👉 [YouTube – Ansible Ad-Hoc Commands (Part 1)](https://youtu.be/R2B_9xBZoJE)

### 📞 Contact Us
**Phone:** [+91 91500 87745](tel:+919150087745)

### 💬 Ask Your Doubts
Join our **Discord Community**  
👉 [Click here to connect](https://discord.gg/N7GBNHBdqw)

### 📺 Explore More Learning
Subscribe to my **YouTube Channel** – *Learn With Mithran*  
🎯 [Watch Now](https://www.youtube.com/@LearnWithMithran)

---

# 🚀 Ansible Part 2 – Advanced Playbook Features

This repository contains all the **Ansible Playbooks** demonstrated in my second YouTube video. It expands on the basics (from Part 1) and covers **registering outputs, verbosity, pre/post tasks, error handling, variables, handlers, and templates**.

## 📚 What You’ll Learn

- ✅ Register and debug module
- ✅ Verbosity levels
- ✅ Pre task & Post task
- ✅ Error Handling
- ✅ Ansible Vault & Variables
- ✅ Notify & Handlers
- ✅ Line in file Module 
- ✅ Jinj2 template module

---

### ✅ 1. Register & Debug Output
Store the output of a command and print it:

```yaml
- hosts: all
  become: yes
  tasks:
    - name: Get date information
      shell: /usr/bin/date
      register: abc      # store output

    - name: Print return information
      ansible.builtin.debug:
        var: abc
```

---

### ✅ 2. Verbosity Levels

Run playbooks with different verbosity:

```bash
ansible-playbook -i inventory.txt myplaybook.yaml      # normal
ansible-playbook -i inventory.txt myplaybook.yaml -v   # verbose
ansible-playbook -i inventory.txt myplaybook.yaml -vv
ansible-playbook -i inventory.txt myplaybook.yaml -vvv
ansible-playbook -i inventory.txt myplaybook.yaml -vvvv
```

You can also control message display with:

```yaml
- hosts: all
  become: yes
  tasks:
  - name: Get date information
    shell: /usr/bin/date
    register: abc      # ! store the output of date in result variable

  - name: Print return information from the previous task
    ansible.builtin.debug:
      var: abc         # ! printing the output of the variable using debug
      verbosity: 2
```

---

### ✅ 3. Pre-tasks & Post-tasks

Execute tasks before and after the main tasks:

```yaml
- hosts: all
  become: yes
  pre_tasks:
    - name: Pre Task 1
      debug: { msg: "Commands in PreTask 1" }
  tasks:
    - name: Task 1
      debug: { msg: "Commands in Task 1" }
  post_tasks:
    - name: Post Task 1
      debug: { msg: "Commands in PostTask 1" }
```

---

### ✅ 4. Error Handling with `block`, `rescue`, `always`

Demonstrates failure handling and recovery:

```yaml
- hosts: all
  remote_user : ec2-user
  become: yes
  tasks:
  - name: Error Handling Example
    block:
      - name: Print a message
        ansible.builtin.debug:
          msg: 'I execute normally'

      - name: Force a failure in middle of recovery! >:-)
        ansible.builtin.command: /bin/false
          
      - name: Never print this
        ansible.builtin.debug:
          msg: 'I never execute, due to the above task failing, :-('
      
    rescue:
      - name: Print when errors
        ansible.builtin.debug:
          msg: 'I caught an error'

      - name: Print when errors
        ansible.builtin.debug:
          msg: 'I caught an error'

      - name: Force a failure in middle of recovery! >:-)
        ansible.builtin.command: /bin/false
        ignore_errors: yes

      - name: Print when errors
        ansible.builtin.debug:
          msg: 'I caught an error'
  
    always:
      - name: Always do this
        ansible.builtin.debug:
          msg: "This always executes"
```

Also includes examples with `ignore_errors: yes`.

---

### ✅ 5. Variables & External Files


sensitive.yaml
```yaml
user: Mithran
pass: Password123
```

Use inline variables and external `vars_files`:

```yaml
- hosts: all
  become: yes
  vars:
    colour: Black
    car: kia
  vars_files:
    sensitive.yaml
  tasks:
    - name: Variables Example
      debug:
        msg: "My fav colour is {{ colour }} and I own a {{ car }} and my username is {{ user }}"
```

---

### ✅ 6. Handlers & Notifications

Trigger a handler when a change occurs:

```yaml
- hosts: all
  become: yes
  vars:
    abc: Arun
  tasks:
    - name: Replace a word in a file
      lineinfile:
        path: /home/ec2-user/content.txt
        regexp: '^Trainer:'
        line: "Trainer: {{ abc }}"
      notify: trigger

  handlers:
    - name: trigger
      debug:
        msg: "Handler executed when there is a change!"
```

---

### ✅ 7. Jinja2 Templates

Render a Jinja2 template on remote hosts:

```yaml
- hosts: all
  become: yes
  vars:
    abc: Mithran
  tasks:
    - name: Template a file
      ansible.builtin.template:
        src: ./sample.xml.j2
        dest: /home/ec2-user/sample.xml
```

---

## 🏃‍♂️ How to Run

1. Update your **inventory file** (e.g., `inventory.txt`) with the target hosts.
2. Execute any playbook:

```bash
ansible-playbook -i inventory.txt playbook_name.yaml
```

3. Add verbosity as needed (`-v`, `-vv`, etc.).

---

## 🔑 Key Takeaways

* Capture command outputs with **register** and view with **debug**.
* Control log details using **verbosity flags**.
* Use **pre\_tasks/post\_tasks** to run code before/after main tasks.
* Handle failures with **block/rescue/always**.
* Manage **variables**, **handlers**, and **templates** for dynamic playbooks.

---

### 📺 Watch the Video

👉 [YouTube – Ansible Vault, Templates & Variables (Part 2)](https://youtu.be/7gKbVXdCPOg)

### 📞 Contact Us
**Phone:** [+91 91500 87745](tel:+919150087745)

### 💬 Ask Your Doubts
Join our **Discord Community**  
👉 [Click here to connect](https://discord.gg/N7GBNHBdqw)

### 📺 Explore More Learning
Subscribe to my **YouTube Channel** – *Learn With Mithran*  
🎯 [Watch Now](https://www.youtube.com/@LearnWithMithran)

---