# ANSIBLE CONFIGURATION MANAGEMENT – AUTOMATE PROJECT 7 TO 10

Scope: configure a Jump server and create an Ansible playbook to automate servers configuration

1. Rename your Jenkins EC2 Instance from prj 10 to Jenkins-Ansible and allocate Elastic IP address

    - install Ansible 

        ```
        sudo apt update
        sudo apt install ansible
        ansible --version # check that it's installed
        ```

2. Create a new repository called `ansible-config-mgt` and a new freestyle project called **ansible** - configure it to point to the ansible-config-mgt repository

3. Set up build trigger with GitHub Webhook 

4. Configure Post build action: save all artifacts (**) 

5. Run `Build now` in Jenkins or commit a change in your Github repository, then check artifacts have been saved in `ls /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/`

6. Clone the repository to your instance: `git clone <ansible-config-mgt repo link>` 

7. Open your repository in VS Code (use git clone)

8. Create a new branch in **ansible-config-mgt** repo, you can name it **feature/prj-11**, then switch into the new branch  

    `git checkout -b <new branch>` #this creates and automatically changes into new branch

9. inside `ansible-config-mgt` create folders and files:

    - `playbooks` - to store all playbook files

        create inside it a file named **common.yml**

    - `inventory` - to store hosts files

        create inside it files **dev.yml, staging.yml, uat.yml and prod.yml** - which are for the Development, Staging Testing, and Production stages respectively

10. Configure ssh-agent for Ansible to ssh into remote servers

    - on local machine run `eval ssh-agent` to start the agent

    - add private key using `ssh-add <private key.pem>`

    - check taht your key has been added using `ssh-add -l`

    `Note` Every time you close the terminal where you started the ssh-agent, the session will end and you will need to start a new one

11. Ssh into your Jenkins-Ansible using ssh-agent- use `ssh -A ubuntu@public-ip` 

    - if you were to try to ssh into a RHEL-based server the user would be `ec2-user`

12. Open `inventory/dev.yml` and paste the following:

    ```
    [nfs]
    <NFS-Server-Private-IP-Address> ansible_ssh_user='ec2-user'

    [webservers]
    <Web-Server1-Private-IP-Address> ansible_ssh_user='ec2-user'
    <Web-Server2-Private-IP-Address> ansible_ssh_user='ec2-user'

    [db]
    <Database-Private-IP-Address> ansible_ssh_user='ubuntu' 

    [lb]
    <Load-Balancer-Private-IP-Address> ansible_ssh_user='ubuntu'
    ```
13. Open `playbooks/common.yml` and paste teh following:

    ```
    ---
    - name: update web and nfs servers
    hosts: webservers, nfs
    remote_user: ec2-user
    become: yes
    become_user: root
    tasks:
        - name: ensure wireshark is at the latest version
        yum:
            name: wireshark
            state: latest

    - name: update LB and db servers
    hosts: lb, db
    remote_user: ubuntu
    become: yes
    become_user: root
    tasks:
        - name: Update apt repo
        apt: 
            update_cache: yes

        - name: ensure wireshark is at the latest version
        apt:
            name: wireshark
            state: latest
    ```

    `Comments` the db server was put at the beginning of the playbook, with nfs and webservers as if it was running on RHEL8 but it was actually running on Ubuntu so the user should have been `ubuntu`. Running the playbook with db server misplaced gave me an error so I made the appropriate changes to teh playbook

    ![error](./screenshots_11/playbook_error.png)

14. Push all changes to your branch, to GitHub

    ```
    git status

    git add <selected files>

    git commit -m "commit message"

    git push
    ```

15. Create a pull request and if no conflicts are found, `merge` the code to the main branch

16. On your terminal checkout to the main branch and pull down all recent changes

    This will trigger a new build and Jenkins will save all the build artifacts to `/var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/` directory on Jenkins-Ansible server.

17. Run the playbook

    ```
    cd ansible-config-mgt
    ansible-playbook -i inventory/dev.yml playbooks/common.yml
    ```
    ![ok](./screenshots_11/run_ok.png)

    Confirm that wireshark was installed on remote servers, and the timezone was changed

    ![wireshark_installed](./screenshots_11/wireshark_version.png)

    ![timezone](./screenshots_11/timezone.png)