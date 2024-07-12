### This playbook can be used to Deploy Secure OpenLDAP cluster with 2 masters in HA by Keepalived on RHEL9 servers with W3 password enabled ###

=========================================================================

This playbook will install and configure slapd,keepalived and saslauthd.

In this example we have below 2 servers  one in Dallas and second one is in Amsterdam


9.46.38.145   daldlldp201a.sand.example.com (Master1)

9.172.62.213  amsdlldp201b.sand.example.com (Master2)

9.46.38.146   dlldp201.sand.example.com (VIP)


### 1.	Prerequisites

a). Put host entry on each master servers in /etc/hosts including VIP. For example

```console
9.172.62.213  amsdlldp201b.sand.example.com
9.46.38.145   daldlldp201a.sand.example.com
9.46.38.146   dlldp201.sand.example.com
```

b). Disable firewalld or allow port 389 and 636 in servers.

c). Configure anton user key and confirm ssh communication from Ansible server to remote hosts

### 2.	Download LDAP-ANSIBLE folder from this GIT to ANSIBLE server  and cd to that folder

```console
[root@pundldpops101 LDAP-ANSIBLE]# ll
total 28
-rw-r--r-- 1 root root  106 Jul  7 04:15 ansible.cfg
-rw------- 1 root root 2622 Jul  7 04:15 antonkey
-rw-r--r-- 1 root root  327 Jul  7 04:15 ldaphosts.inv
-rw-r--r-- 1 root root  333 Jul  7 04:15 ldapvars
-rw-r--r-- 1 root root 7218 Jul  7 04:15 ldap.yaml
drwxr-xr-x 3 root root 4096 Jul  7 04:16 openldap-master
[root@pundldpops101 LDAP-ANSIBLE]# 
```

a). Update the file ldapvars as per your requirement

The searchbase of a search is the starting point. Where it will start searching. Pretty self-explanatory.

The binddn DN is basically the credential you are using to authenticate against an LDAP.When using a bindDN it usually comes with a password associated with it. Here we use manager user with manager password.

b). Update the file openldap-master/base.ldif.j2 with required user dn and group dn which depends on your LDAP cluster naming convention and use.
Dont change already defined variables. Change only "ou" details according to the region and project.

c). Copy the certificate and key inside the folder openldap-master for LDAPS set up

Server certificate (tls.pem) key (tls.key) ca cert (digicert_ca.pem) - Use same naming convention

In this example I used certs for *.sand.example.com (wild card)

```console
[root@pundldpops101 LDAP-ANSIBLE]# ls -al openldap-master/ | grep key
-rw-r--r-- 1 root root  3243 Jul  7 04:15 tls.key
[root@pundldpops101 LDAP-ANSIBLE]# ls -al openldap-master/ | grep pem
-rw-r--r-- 1 root root  3014 Jul  7 04:15 digicert_ca.pem
-rw-r--r-- 1 root root  2841 Jul  7 04:15 tls.pem
[root@pundldpops101 LDAP-ANSIBLE]# 
```

d). Update inventory host file with anton key and remote server details. Refer below example

```console
[root@pundldpops101 LDAP-ANSIBLE]# cat ldaphosts.inv
#change inventory as per the requirement
[master1]
9.46.38.145  ansible_user=anton ansible_ssh_private_key_file=/home/smanikka/LDAP-ANSIBLE/antonkey
[master2]
9.172.62.213  ansible_user=anton ansible_ssh_private_key_file=/home/smanikka/LDAP-ANSIBLE/antonkey
```
e). Take VM snapshot from Vcenter before starting installation so that we can revert it easily if anything didnt work,  fix the playbook and rerun.

### 3.	We need to generate a password for LDAP Manager and configuration replication. Put the details in ldapvars file with original password and encrypted one.

```console
slappasswd
New password:
Re-enter new password:
{SSHA}MAfw/QNizKx4NxueW7CpCSN6jeDB5Z+C
```
### 4. Run the playbook with below command from the Folder LDAP-ANSIBLE

```console

[root@pundldpops101 LDAP-ANSIBLE]# ansible-playbook -i ldaphosts.inv ldap.yaml  --extra-vars "@ldapvars" -v
Using /opt/LDAP-ANSIBLE/ansible.cfg as config file
[DEPRECATION WARNING]: COMMAND_WARNINGS option, the command warnings feature is being removed. This feature will be removed from ansible-core in version 2.14. Deprecation warnings can be disabled by setting deprecation_warnings=False in 
ansible.cfg.
[WARNING]: Skipping plugin (/usr/local/lib/python3.6/site-packages/ansible/plugins/callback/selective.py) as it seems to be invalid: cannot import name 'codeCodes'
[WARNING]: ansible.utils.display.initialize_locale has not been called, this may result in incorrectly calculated text widths that can cause Display to print incorrect line lengths

PLAY [all] ******************************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************************************************************************************************************
[DEPRECATION WARNING]: Distribution rhel 9.4 on host 9.172.62.213 should use /usr/libexec/platform-python, but is using /usr/bin/python for backward compatibility with prior Ansible releases. A future Ansible release will default to using the 
discovered platform python for this host. See https://docs.ansible.com/ansible-core/2.11/reference_appendices/interpreter_discovery.html for more information. This feature will be removed in version 2.12. Deprecation warnings can be disabled by
 setting deprecation_warnings=False in ansible.cfg.
ok: [9.172.62.213]
[DEPRECATION WARNING]: Distribution rhel 9.4 on host 9.46.38.145 should use /usr/libexec/platform-python, but is using /usr/bin/python for backward compatibility with prior Ansible releases. A future Ansible release will default to using the 
discovered platform python for this host. See https://docs.ansible.com/ansible-core/2.11/reference_appendices/interpreter_discovery.html for more information. This feature will be removed in version 2.12. Deprecation warnings can be disabled by
 setting deprecation_warnings=False in ansible.cfg.
ok: [9.46.38.145]

*****************
****************


PLAY RECAP ******************************************************************************************************************************************************************************************************************************************
9.172.62.213               : ok=34   changed=31   unreachable=0    failed=0    skipped=8    rescued=0    ignored=0   
9.46.38.145                : ok=40   changed=37   unreachable=0    failed=0    skipped=2    rescued=0    ignored=1   

```
### 5. You can access LDAP cluster with VIP hostname on secure port via JXPLORER, and confirm.
