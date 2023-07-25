# ANSIBLE REFACTORING AND STATIC ASSIGNMENTS (IMPORTS AND ROLES)

Scope: refactor code in ansible-config-mgt, create roles for web servers and implement Ansible imports


1. In your `Jenkins-Ansible` instance create a new directory and change permissions 

    ```
    mkdir /home/ubuntu/ansible-config-artifact
    chmod -R 0777 /home/ubuntu/ansible-config-artifact
    ```


2. Instal the **Copy artifact** plugin in Jenkins - without restart


3. Create a new freestyle project named **save_artifacts** and configure it to trigger a job after completion of a build in the `ansible` project

    - Under `General` select `Discard old buildS` and enter a number of max builds to keep e.g. 2

    - Enable the build trigger to `Build after other projects are built` and enter `ansible` under projects to watch


4.  Under `Build steps` select `Copy artifacts from another project` 
    
     -  Specify `**` to copy all artifacts,  `ansible` as a source project and `/home/ubuntu/ansible-config-artifact` as a target directory

     This will save all build artifacts into the ansible-config-artifact folder


5. Test your configuration (click on `Build now` or commit changes) and make sure that files are saved into ansible-config-artifact target directory

    ![buil](./screenshots_12/build.png)

6. Create a new branch named **refactor** and pull down the latest changes from the **main** branch

    - also pull down latest changes for **main** branch in your terminal


7. Checkout to the **refactor** branch and inside **playbooks** folder create a new file named **site.yml** (this will be parent to all other playbooks that will be developed)


8. Create a new folder `mkdir /home/ubuntu/ansible-config-mgt/static_assignments` - this is where all children playbooks will be stored


9. Move **common.yml** file into the newly created **static-assignments** folder 

    -  also create a new file **common-del.yml** inside this folder and paste this into it

        ```
        # delete wireshark
        ---
        - name: update web, nfs and db servers
        hosts: webservers, nfs, db
        remote_user: ec2-user
        become: yes
        become_user: root
        tasks:
        - name: delete wireshark
            yum:
            name: wireshark
            state: removed

        - name: update LB server
        hosts: lb
        remote_user: ubuntu
        become: yes
        become_user: root
        tasks:
        - name: delete wireshark
            apt:
            name: wireshark-qt
            state: absent
            autoremove: yes
            purge: yes
            autoclean: yes
        ```


10. Inside **site.yml** paste the following

     ```
    ---
    - hosts: all
    - import_playbook: ../static-assignments/common-del.yml
    ```


11. Run **site.yml** against the **dev** servers

    ```
    cd /home/ubuntu/ansible-config-mgt/

    ansible-playbook -i inventory/dev.yml playbooks/site.yaml
    ```

    `Note` remember to start ssh-agent and add your private key before running the playbook,  then check that **wireshark** has been deleted using `wireshark --version`


12. Launch two new EC2 instances based on RHEL 8 - **Web1-UAT** and **Web2-UAT**


13. Use `Ansible-galaxy` utility to create a role folder the with default structure

    ```
    mkdir roles
    cd roles
    ansible-galaxy init webserver
    ```

    - once created you can delete folders that are not needed such as `tests, files and vars` 

        ```
        cd /roles/webserver
        sudo rm -r tests files vars
        ```


14. Update the **uat.yml** file inside **inventory** with IP addresses of your 2 UAT Web servers

    ```
    [uat-webservers]
    <Web1-UAT-Server-Private-IP-Address> ansible_ssh_user='ec2-user' 

    <Web2-UAT-Server-Private-IP-Address> ansible_ssh_user='ec2-user' 
    ```

15. In `/etc/ansible/ansible.cfg` file uncomment `roles_path` string and provide a full path to your roles directory 

    `roles_path    = /home/ubuntu/ansible-config-mgt/roles`, so Ansible could know where to find configured roles


16. Inside `roles/webserver/tasks/main.yml` paste this

    ```
    ---
    - name: install apache
    become: true
    ansible.builtin.yum:
        name: "httpd"
        state: present

    - name: install git
    become: true
    ansible.builtin.yum:
        name: "git"
        state: present

    # insert link to tooling repository 
    - name: clone a repo
    become: true
    ansible.builtin.git:
        repo: https://github.com/adaanene/tooling-1.git
        dest: /var/www/html
        force: yes

    - name: copy html content to one level up
    become: true
    command: cp -r /var/www/html/html/ /var/www/

    - name: Start service httpd, if not started
    become: true
    ansible.builtin.service:
        name: httpd
        state: started

    - name: recursively remove /var/www/html/html/ directory
    become: true
    ansible.builtin.file:
        path: /var/www/html/html
        state: absent
    ```


17. Inside **static_assignments** create a file `touch uat-webservers.yml`

    - paste into the new file

        ```
        ---
        - hosts: uat-webservers
        roles:
        - webserver
        ```


18. Update **site.yml** so it also refers to **uat-webserver.yml**

    ```
    # site.yml should look like this
    ---
    - hosts: all
    - import_playbook: ../static-assignments/common.yml

    - hosts: uat-webservers
    - import_playbook: ../static-assignments/uat-webservers. yml
    ```

19. Do the following:

    - commit changes to `refactor` branch
    - create a pull request and merge with `main` branch
    - make sure webhook triggered jobs in Jenkins and copied files to `/home/ubuntu/ansible-config-mgt/` directory


20. Run the playbook again `uat` servers

    `sudo ansible-playbook -i /home/ubuntu/ansible-config-mgt/inventory/uat.yml /home/ubuntu/ansible-config-mgt/playbooks/site.yaml`

    `Note` remember to start ssh-agent and add key

21. Your uat servers have now been configured and you can reach them from your browser

    ```
    http://<Web1-UAT-Server-Public-IP-or-Public-DNS-Name>/index.php

    http://<Web1-UAT-Server-Public-IP-or-Public-DNS-Name>/index.php
    ```

    `Note` remember to create inbound rule for HTTP connection on port 80