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

## 📺 Watch the Video

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

## 📺 Watch the Video

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

# 🚀 Part 3 – Ansible Gather Facts

This README is part of the **hands-on Ansible learning series**. In this section, you’ll practice:

* Gathering and using **Ansible Facts**
* Running tasks conditionally based on facts
* Advanced playbook features like **local actions, async jobs, background tasks, Apache deployment, and roles**

## 📚 What You’ll Learn

* ✅ What are Ansible Facts
* ✅ Gathering and filtering facts
* ✅ Using facts for conditional execution
* ✅ Running local vs remote tasks
* ✅ Running async tasks with polling
* ✅ Running cunrrent jobs with `poll: 0`
* ✅ Installing and configuring Apache webserver as Roles
* ✅ Organizing playbooks with roles

---

## 📚 What are Ansible Facts?

* Facts are **system-level information** like hostname, IP, OS type, CPU, memory, etc.
* Ansible collects these automatically at the start of playbook execution (unless disabled).
* You can use facts to write **dynamic playbooks** that adapt to different environments.

---

### ⚡ 1️⃣ Gather Facts with Ad-Hoc Command

Use the `setup` module to collect and display all available facts:

```bash
ansible all -i slaves.txt -m setup
```

Filter only specific facts:

```bash
ansible all -i slaves.txt -m setup -a "filter=ansible_hostname"
ansible all -i slaves.txt -m setup -a "filter=ansible_os_family"
```

---

### 🖥️ 2️⃣ Playbook – Print All Facts

You can print facts directly inside a playbook:

```yaml
- hosts: all
  remote_user: ec2-user
  become: yes
  tasks:
    - name: Print all available facts
      ansible.builtin.debug:
        var: ansible_facts
```

---

### 🏷️ 3️⃣ Playbook – Conditional Task Using Facts

Run a task only on systems that match a specific OS family:

```yaml
- hosts: all
  remote_user: ec2-user
  become: yes
  tasks:
    - name: Shut down Amazon flavored systems
      ansible.builtin.command: /sbin/shutdown -t now
      when: ansible_facts['os_family'] == "Amazon"
```

---

### 🔄 4️⃣ Multiple Conditional Tasks

You can define different tasks for different OS families using `when`:

```yaml
- hosts: all
  become: yes
  tasks:
    - name: 1st Task
      debug:
        msg: Running 1st Task
      when: ansible_facts['os_family'] == "RedHat"

    - name: 2nd Task
      debug:
        msg: Running 2nd Task
      when: ansible_facts['os_family'] == "Ubuntu"

    - name: 3rd Task
      debug:
        msg: Running 3rd Task
      when: ansible_facts['os_family'] == "RedHat"
```

---

### ⚡ 5️⃣ delegate_to & Local Action Example

```yaml
- hosts: all
  become: yes
  tasks:
    - name: 1st Task
      debug:
        msg: Running 1st Task
      delegate_to: 10.0.0.1

    - name: 2nd task (local execution)
      local_action: ansible.builtin.command touch xyz.txt

    - name: 3rd Task
      debug:
        msg: Running 3rd Task
```

---

## ⚡ Ansible: Serial and Fork Behavior

This document explains how **Ansible executes tasks** across multiple hosts using the **`serial`** and **`forks`** parameters, along with examples demonstrating execution time under different scenarios.

---

### 1️⃣ Introduction

Ansible executes tasks across hosts in parallel by default, but you can control execution behavior using:

* **`serial`**: Limit the number of hosts to act on at a time. Tasks are executed on `serial` number of hosts, then moves to the next batch.
* **`forks`**: Number of parallel processes Ansible spawns to execute tasks. Controls how many hosts run simultaneously.
* **`async` and `poll`**: Run tasks asynchronously. `poll: 0` makes tasks fire-and-forget, allowing faster overall execution.

---

### 2️⃣ Normal Execution Behavior

By default, Ansible runs tasks sequentially on all hosts (limited by forks).

**Example (6 worker nodes, 2 tasks, 5 sec each):**

```
TASK1: ABCDEF -> 5 sec
TASK2: ABCDEF -> 5 sec
Total time: 10 sec
```

```yaml
- hosts: all
  become: yes
  tasks:
    - name: run for 5 seconds
      ansible.builtin.command: /bin/sleep 5

    - name: run for 5 seconds
      ansible.builtin.command: /bin/sleep 5
```

---

### 3️⃣ Async Tasks (`poll: 0`)

When a task is run asynchronously with `poll: 0`, Ansible fires tasks on all hosts and does not wait for completion.

**Example (6 worker nodes, 2 tasks, 5 sec each, async poll=0):**

```
TASK1: ABCDEF -> 5 sec
TASK2: ABCDEF -> 5 sec
Total time: 5 sec
```

```yaml
- hosts: all
  become: yes
  tasks:
    - name: run for 5 seconds
      ansible.builtin.command: /bin/sleep 5
      poll: 0

    - name: run for 5 seconds
      ansible.builtin.command: /bin/sleep 5
      poll: 0
```

> Key takeaway: Async tasks allow overlapping execution, reducing total time.

---

### 4️⃣ Serial Execution

`serial` allows tasks to run on a subset of hosts in batches.

**Example (2 tasks, 2 worker nodes, serial=1):**

```
Group1: A
  Task1: 3 sec
  Task2: 7 sec
Group2: B
  Task1: 3 sec
  Task2: 7 sec
Total time: 10 sec
```

> Key takeaway: Serial executes tasks in batches sequentially, so total time depends on batch size.

---

### 5️⃣ Serial with Multiple Hosts & Tasks

**Example (3 tasks, 6 hosts, serial=2, 5 sec each task):**

```
Batch1: AB
  Task1: 5 sec
  Task2: 5 sec
  Task3: 5 sec
Batch2: CD
  Task1: 5 sec
  Task2: 5 sec
  Task3: 5 sec
Batch3: EF
  Task1: 5 sec
  Task2: 5 sec
  Task3: 5 sec
Total time: 45 sec
```

> Key takeaway: Serial controls host concurrency; tasks complete per batch before moving to next.

---

### 6️⃣ Forks (Parallelism within Serial Batches)

`forks` controls how many hosts Ansible executes on simultaneously within each batch.

**Example 1 (2 tasks, 4 hosts, serial=4, fork=2, 5 sec per task):**

```
First Run: ABCD
  Task1: AB 5 sec + CD 5 sec
  Task2: AB 5 sec + CD 5 sec
Total time: 20 sec
```

**Example 2 (2 tasks, 10 hosts, serial=4, fork=3, 5 sec per task):**

```
First Run: ABCD
  Task1: ABC 5 sec + D 5 sec -> 10 sec
  Task2: ABC 5 sec + D 5 sec -> 10 sec
Second Run: EFGH
  Task1: EFG 5 sec + H 5 sec -> 10 sec
  Task2: EFG 5 sec + H 5 sec -> 10 sec
Third Run: IJ
  Task1: IJ 5 sec
  Task2: IJ 5 sec
Total time: 50 sec
```

> Key takeaway: Forks allow further parallelism within the serial batch. Total time depends on serial batch size and fork count.

---

### 7️⃣ Key Takeaways

1. **Default Execution**: Tasks run sequentially across hosts.
2. **Async + poll=0**: Executes tasks on all hosts simultaneously, reducing total runtime.
3. **Serial**: Executes tasks in batches of hosts. Smaller batch → longer total time.
4. **Forks**: Enables parallel execution within each batch. Increasing forks reduces runtime.
5. **Combined Effect**: Total execution time is determined by tasks × batch size × forks.

---

### 8️⃣ Visual Summary

| Scenario         | Hosts | Tasks | Serial | Forks | Total Time |
| ---------------- | ----- | ----- | ------ | ----- | ---------- |
| Normal           | 6     | 2     | -      | -     | 10 sec     |
| Async poll=0     | 6     | 2     | -      | -     | 5 sec      |
| Serial=1         | 2     | 2     | 1      | -     | 10 sec     |
| Serial=2         | 6     | 3     | 2      | -     | 45 sec     |
| Serial=4, Fork=2 | 4     | 2     | 4      | 2     | 20 sec     |
| Serial=4, Fork=3 | 10    | 2     | 4      | 3     | 50 sec     |

---


## 📚 Ansible Roles

Roles help organize tasks, handlers, files, templates, and variables into a structured directory format, making playbooks modular and reusable.

---

### 🗂️ Role Directory Structure

When converted into a role (`apache`), the structure looks like this:

```
roles/
└── apache/
    ├── tasks/
    │   └── main.yml
    ├── files/
    │   └── index.html
    ├── handlers/
    │   └── main.yml
    ├── vars/
    │   └── main.yml
    └── defaults/
        └── main.yml
```

* **tasks/main.yml** → contains the main tasks.
* **files/** → static files like `index.html` that need to be copied.
* **handlers/main.yml** → restart services if required.
* **vars/** and **defaults/** → define variables.

---

### 📦  Using Roles in Playbooks

```yaml
- hosts: all
  become: yes
  roles:
    - apache
  tasks:
    - name: Roll Called in my Playbook
      debug:
        msg: Playbook Executed
```

---

### ✅ Key Takeaways

* **Roles** make playbooks modular, reusable, and easier to maintain.
* **Async + poll** lets tasks run without blocking execution.
* **Poll: 0** allows fire-and-forget background jobs.
* **Serial** ensures safe rolling updates on a limited number of hosts.
* **Forks** control concurrency for scaling across large inventories.

---

# 🚀 Ansible Task Time Calculations

Assume each task takes **5 seconds** to run.

## 1️⃣ TASK: 4 Tasks, 6 nodes, serial = 3

<details>
<summary>Click to reveal answer</summary>

* Total nodes: 6
* Serial = 3 → max 3 nodes run at a time
* Each task takes 5 seconds

**Calculation:**

1. First batch of 3 nodes → 5 sec
2. Second batch of 3 nodes → 5 sec

* Each task runs sequentially across batches → 4 tasks × 10 sec = **40 sec**

**Answer:** 40 seconds

</details>

---

## 2️⃣ TASK: 3 Tasks, 10 nodes, forks = 5

<details>
<summary>Click to reveal answer</summary>

* Forks = 5 → max 5 nodes run in parallel
* Each task takes 5 sec

**Calculation:**

1. 10 nodes / 5 forks = 2 batches per task
2. Each batch = 5 sec → per task = 10 sec
3. 3 tasks sequentially → 3 × 10 sec = **30 sec**

**Answer:** 30 seconds

</details>

---

## 3️⃣ TASK: 2 Tasks, 10 nodes, forks = 5, serial = 4

<details>
<summary>Click to reveal answer</summary>

* Serial = 4 → only 4 nodes run at a time
* Forks = 5 → max 5 parallel tasks per play
* Each task takes 5 sec

**Calculation:**

1. 10 nodes / serial 4 → 3 batches (4+4+2 nodes)
2. Each batch = 5 sec

* Each task = 15 sec
* 2 tasks sequentially → 2 × 15 = **30 sec**

**Answer:** 30 seconds

</details>

---

## 4️⃣ TASK: 3 Tasks, 8 nodes, forks = 3, serial = 2

<details>
<summary>Click to reveal answer</summary>

* Serial = 2 → 2 nodes at a time
* Forks = 3 → ignored if serial < forks
* Each task takes 5 sec

**Calculation:**

1. 8 nodes / serial 2 → 4 batches per task
2. Each batch = 5 sec → per task = 20 sec
3. 3 tasks sequentially → 3 × 20 sec = **60 sec**

**Answer:** 60 seconds

</details>

---

## 📺 Watch the Video

👉 [YouTube – Ansible Roles, Gather Facts & Task Execution Order (Part 3)](https://youtu.be/1IEhp-Pv-vc)

### 📞 Contact Us
**Phone:** [+91 91500 87745](tel:+919150087745)

### 💬 Ask Your Doubts
Join our **Discord Community**  
👉 [Click here to connect](https://discord.gg/N7GBNHBdqw)

### 📺 Explore More Learning
Subscribe to my **YouTube Channel** – *Learn With Mithran*  
🎯 [Watch Now](https://www.youtube.com/@LearnWithMithran)

---