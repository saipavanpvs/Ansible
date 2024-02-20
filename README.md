# Ansible
Building an inventory
Inventories organize managed nodes in centralized files that provide Ansible with system information and network locations. Using an inventory file, Ansible can manage a large number of hosts with a single command.

To complete the following steps, you will need the IP address or fully qualified domain name (FQDN) of at least one host system. For demonstration purposes, the host could be running locally in a container or a virtual machine. You must also ensure that your public SSH key is added to the authorized_keys file on each host.

Continue getting started with Ansible and build an inventory as follows:

Create a file named inventory.ini in the ansible_quickstart directory that you created in the preceding step.

Add a new [myhosts] group to the inventory.ini file and specify the IP address or fully qualified domain name (FQDN) of each host system.

[myhosts]
192.0.2.50
192.0.2.51
192.0.2.52
Verify your inventory.

ansible-inventory -i inventory.ini --list
Ping the myhosts group in your inventory. 
-----------------------------------------------------
Pinging hosts in a configuration management context, particularly with tools like Ansible, is often used as a basic connectivity test to verify that the hosts are reachable and responsive. It's a simple way to check the network connectivity and ensure that the hosts can be communicated with before more complex tasks are executed. The ping operation typically involves sending a small packet of data to the target host and waiting for a response.


ansible myhosts -m ping -i inventory.ini
Note

Pass the -u option with the ansible command if the username is different on the control node and the managed node(s).


--------------------------------------------------------------------------------------------------------------------------------------

Tips for building inventories
Ensure that group names are meaningful and unique. Group names are also case sensitive.

Avoid spaces, hyphens, and preceding numbers (use floor_19, not 19th_floor) in group names.

Group hosts in your inventory logically according to their What, Where, and When.

What
Group hosts according to the topology, for example: db, web, leaf, spine.

Where
Group hosts by geographic location, for example: datacenter, region, floor, building.

When
Group hosts by stage, for example: development, test, staging, production.


whenever in ansible if a particular tasks fail, the subsequent task will be a failure and playbook will be exited.
To avoid this, use a tag ignore_errors: yes, it will ignore the current task errros and proceed with the next step.

gather_facts: yes or no

by default, it is yes, if it is set to no, ansible will not gather the facts related to the hosts.

Note: gathering facts means getting all the information about the hosts assigned.

--> ansible provides all the builtin modules or controllers to access any of the commands.
if it is not in ansible, we can use shell commands, by default ansible shell won't print the output of the shell result.
to get the output, register is used, it will capture the output by the shell commands.

set_facts:
It will capture the output of the result and store it in a varibale.
it can be used anywhere.
It can be used to capture the error codes and based on it , we can perform actions on it.




In Ansible, both register and set_fact are used to store data during the execution of a playbook, but they serve different purposes.

register: This is used to capture the result of a task (or a command) into a variable. It's often used with tasks that involve executing commands, and the captured data can be referenced later in the playbook.

Example:

yaml
Copy code
- name: Run a command and register the output
  ansible.builtin.shell: echo "Hello, World!"
  register: command_output

- name: Print the registered variable
  ansible.builtin.debug:
    msg: "{{ command_output.stdout }}"
In this example, the output of the echo command is captured in the command_output variable, and later it's printed using ansible.builtin.debug.

set_fact: This module is used to set the value of a variable directly in the playbook. It allows you to define variables dynamically during playbook execution.

Example:

yaml
Copy code
- name: Set a variable using set_fact
  ansible.builtin.set_fact:
    my_variable: "This is a value"

- name: Print the variable set using set_fact
  ansible.builtin.debug:
    msg: "{{ my_variable }}"
In this example, the my_variable is set using set_fact, and then it's printed using ansible.builtin.debug.

In summary, register is typically used to capture the output of commands or tasks, while set_fact is used to set the value of variables directly within the playbook. The choice between them depends on the specific use case and what you want to achieve in your playbook.


User
why to use setfact , even we can assign a value directly to a variable right
ChatGPT
Yes, you are correct that you can assign values directly to variables in Ansible without using set_fact. However, the use of set_fact becomes valuable when you need to set or modify variables dynamically during the playbook execution, especially when the values depend on the results of previous tasks or facts discovered about the system.

Here are a few scenarios where set_fact can be beneficial:

Dynamic Value Assignment:

yaml
Copy code
- name: Set variable dynamically based on a condition
  ansible.builtin.set_fact:
    my_variable: "{{ 'value1' if some_condition else 'value2' }}"
You can dynamically assign values based on conditions or the output of previous tasks.

Calculated Values:

yaml
Copy code
- name: Calculate a value and set it as a variable
  ansible.builtin.set_fact:
    calculated_value: "{{ some_number * 2 }}"
You can perform calculations and set the result as a variable.

Derived Facts:

yaml
Copy code
- name: Set variable based on a fact about the system
  ansible.builtin.set_fact:
    operating_system: "{{ ansible_facts['distribution'] }}"
You can set variables based on facts about the system discovered during the playbook execution.

While it's true that you can assign values directly to variables, set_fact provides more flexibility and allows you to create dynamic, context-dependent playbooks. It can be particularly useful in scenarios where the values depend on the runtime context or are derived from other tasks' outputs or facts.


reference:https://chat.openai.com/c/7d4af45e-cb0f-4915-94a1-621f0db4d037


=============Blocks & Rescue ========================

Blocks
Blocks create logical groups of tasks. Blocks also offer ways to handle task errors, similar to exception handling in many programming languages.

Grouping tasks with blocks

Handling errors with blocks

Grouping tasks with blocks
All tasks in a block inherit directives applied at the block level. Most of what you can apply to a single task (with the exception of loops) can be applied at the block level, so blocks make it much easier to set data or directives common to the tasks. The directive does not affect the block itself, it is only inherited by the tasks enclosed by a block. For example, a when statement is applied to the tasks within a block, not to the block itself.

Block example with named tasks inside the block
 tasks:
   - name: Install, configure, and start Apache
     block:
       - name: Install httpd and memcached
         ansible.builtin.yum:
           name:
           - httpd
           - memcached
           state: present

       - name: Apply the foo config template
         ansible.builtin.template:
           src: templates/src.j2
           dest: /etc/foo.conf

       - name: Start service bar and enable it
         ansible.builtin.service:
           name: bar
           state: started
           enabled: True
     when: ansible_facts['distribution'] == 'CentOS'
     become: true
     become_user: root
     ignore_errors: true
In the example above, the ‘when’ condition will be evaluated before Ansible runs each of the three tasks in the block. All three tasks also inherit the privilege escalation directives, running as the root user. Finally, ignore_errors: true ensures that Ansible continues to execute the playbook even if some of the tasks fail.

Note

All tasks in a block, including the ones included through include_role, inherit directives applied at the block level.

Names for blocks have been available since Ansible 2.3. We recommend using names in all tasks, within blocks or elsewhere, for better visibility into the tasks being executed when you run the playbook.

Handling errors with blocks
You can control how Ansible responds to task errors using blocks with rescue and always sections.

Rescue blocks specify tasks to run when an earlier task in a block fails. This approach is similar to exception handling in many programming languages. Ansible only runs rescue blocks after a task returns a ‘failed’ state. Bad task definitions and unreachable hosts will not trigger the rescue block.

Block error handling example
 tasks:
   - name: Handle the error
     block:
       - name: Print a message
         ansible.builtin.debug:
           msg: 'I execute normally'

       - name: Force a failure
         ansible.builtin.command: /bin/false

       - name: Never print this
         ansible.builtin.debug:
           msg: 'I never execute, due to the above task failing, :-('
     rescue:
       - name: Print when errors
         ansible.builtin.debug:
           msg: 'I caught an error, can do stuff here to fix it, :-)'
You can also add an always section to a block. Tasks in the always section run no matter what the task status of the previous block is.

Block with always section
 tasks:
   - name: Always do X
     block:
       - name: Print a message
         ansible.builtin.debug:
           msg: 'I execute normally'

       - name: Force a failure
         ansible.builtin.command: /bin/false

       - name: Never print this
         ansible.builtin.debug:
           msg: 'I never execute :-('
     always:
       - name: Always do this
         ansible.builtin.debug:
           msg: "This always executes, :-)"
Together, these elements offer complex error handling.

Block with all sections
 tasks:
   - name: Attempt and graceful roll back demo
     block:
       - name: Print a message
         ansible.builtin.debug:
           msg: 'I execute normally'

       - name: Force a failure
         ansible.builtin.command: /bin/false

       - name: Never print this
         ansible.builtin.debug:
           msg: 'I never execute, due to the above task failing, :-('
     rescue:
       - name: Print when errors
         ansible.builtin.debug:
           msg: 'I caught an error'

       - name: Force a failure in middle of recovery! >:-)
         ansible.builtin.command: /bin/false

       - name: Never print this
         ansible.builtin.debug:
           msg: 'I also never execute :-('
     always:
       - name: Always do this
         ansible.builtin.debug:
           msg: "This always executes"
The tasks in the block execute normally. If any tasks in the block return failed, the rescue section executes tasks to recover from the error. The always section runs regardless of the results of the block and rescue sections.

If an error occurs in the block and the rescue task succeeds, Ansible reverts the failed status of the original task for the run and continues to run the play as if the original task had succeeded. The rescued task is considered successful and does not trigger max_fail_percentage or any_errors_fatal configurations. However, Ansible still reports a failure in the playbook statistics.

You can use blocks with flush_handlers in a rescue task to ensure that all handlers run even if an error occurs:

Block run handlers in error handling
 tasks:
   - name: Attempt and graceful roll back demo
     block:
       - name: Print a message
         ansible.builtin.debug:
           msg: 'I execute normally'
         changed_when: true
         notify: Run me even after an error

       - name: Force a failure
         ansible.builtin.command: /bin/false
     rescue:
       - name: Make sure all handlers run
         meta: flush_handlers
 handlers:
    - name: Run me even after an error
      ansible.builtin.debug:
        msg: 'This handler runs even on error'
New in version 2.1.

Ansible provides a couple of variables for tasks in the rescue portion of a block:

ansible_failed_task
The task that returned ‘failed’ and triggered the rescue. For example, to get the name use ansible_failed_task.name.

ansible_failed_result
The captured return result of the failed task that triggered the rescue. This would equate to having used this var in the register keyword.

These can be inspected in the rescue section:

Use special variables in rescue section.
 tasks:
   - name: Attempt and graceful roll back demo
     block:
       - name: Do something
         ansible.builtin.shell: grep $(whoami) /etc/hosts

       - name: Force a failure, if previous one succeeds
         ansible.builtin.command: /bin/false
     rescue:
       - name: All is good if the first task failed
         when: ansible_failed_task.name == 'Do Something'
         ansible.builtin.debug:
           msg: All is good, ignore error as grep could not find 'me' in hosts

       - name: All is good if the first task failed
         when: "'/bin/false' in ansible_failed_result.cmd | d([])"
         ansible.builtin.fail:
           msg: It is still false!!!


####### RABBITMQ USER CREATION #########

- name: Create RabbitMQ User
  community.rabbitmq.rabbitmq_user:
    user: joe
    password: "securepassword"  # Replace with a strong and secure password
    vhost: /                     # Specify the virtual host
    configure_priv: ".*"         # Permissions to configure resources
    read_priv: ".*"              # Permissions to read from resources
    write_priv: ".*"             # Permissions to write (publish) to resources
    state: present

configure_priv: .*: Grants the user permission to configure (create, delete, or modify) any resource within the specified virtual host.

read_priv: .*: Grants the user permission to read (consume messages) from any resource within the specified virtual host.

write_priv: .*: Grants the user permission to write (publish messages) to any resource within the specified virtual host.


It seems like your input is a forward slash ("/"), which is often used to represent the root directory or the root of a hierarchy. In the context of RabbitMQ, the forward slash is commonly used to denote the default virtual host.

In the previous examples related to RabbitMQ, when specifying vhost: /, it means that you are referring to the root virtual host. In RabbitMQ, a virtual host is a way to partition resources and permissions within the RabbitMQ broker.

Here's a quick recap:

vhost: /: Specifies the root virtual host. In RabbitMQ, the root virtual host is typically used by default, and it is denoted by a forward slash ("/").
So, if you are creating a RabbitMQ user with permissions for the root virtual host, using vhost: / is appropriate. If you have additional virtual hosts configured in your RabbitMQ setup, you would replace / with the name of the specific virtual host you want the user to have access to.


In the context of message-oriented middleware systems like RabbitMQ, a virtual host is a way to create multiple isolated environments within a single RabbitMQ broker. Each virtual host has its own set of exchanges, queues, and bindings, as well as its own users and permissions. This allows you to logically separate different applications or clients, providing a level of isolation and organization.


