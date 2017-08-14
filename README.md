# file lookup issue


when using lookup(file, "") the lookup fails when done as a local_action

this is tested using lineinfile, but also fails to lookup for (eg) addng ssh keys

results:

expected: both local and remote work

actual: remote works but local fails with
[WARNING]: Unable to find 'user1.pub' in expected paths.



NOTE2: works with older ansible 2.1.1 but fails with 2.3.2


reproduce:

 $ cd playbooks/
 $ chmod +x ssh-test.yml
 $ ./ssh-test.yml



 $ ansible --version
 ansible 2.3.2.0
   config file = /etc/ansible/ansible.cfg
   configured module search path = Default w/o overrides
   python version = 2.7.12 (default, Nov 19 2016, 06:48:10) [GCC 5.4.0 20160609]


 playbooks $ ./ssh-test.yml
SUDO password:

PLAY [172.16.176.142] **************************************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************************************
ok: [172.16.176.142]

TASK [doSSH : LOCAL blank temp file] ***********************************************************************************************************************************
changed: [172.16.176.142]

TASK [doSSH : REMOTE blank temp file] **********************************************************************************************************************************
ok: [172.16.176.142 -> localhost]

TASK [doSSH : Write users to temp REMOTE file] *************************************************************************************************************************
changed: [172.16.176.142] => (item={u'key': u'user1 line one\nuser1 line two', u'grp_ops': True, u'name': u'user1', u'groups': u'ops'})
changed: [172.16.176.142] => (item={u'key': u'user two, line 1\nuser two, line 2\nuser two, line 3', u'grp_ops': True, u'name': u'user2', u'groups': u'ops'})
changed: [172.16.176.142] => (item={u'key': u'ssh-rsa AAABBBCCC sshdevuser@nogroup', u'name': u'devuser', u'groups': u'dev'})
 [WARNING]: Unable to find 'user1.pub' in expected paths.

ERROR! [{u'groups': u'ops', u'grp_ops': True, u'name': u'user1', u'key': u"{{ lookup('file', 'user1.pub') }}"}, {u'groups': u'ops', u'grp_ops': True, u'name': u'user2', u'key': u"{{ lookup('file', 'user2.pub') }}"}, {u'groups': u'dev', u'name': u'devuser', u'key': u'ssh-rsa AAABBBCCC sshdevuser@nogroup'}]: An unhandled exception occurred while running the lookup plugin 'file'. Error was a <class 'ansible.errors.AnsibleError'>, original message: could not locate file in lookup: user1.pub


----------------------------------

playbooks $ ansible --version
ansible 2.1.1.0
  config file = /etc/ansible/ansible.cfg
  configured module search path = Default w/o overrides


playbooks $ ./ssh-test.yml
SUDO password:

PLAY [172.16.176.142] **********************************************************

TASK [setup] *******************************************************************
ok: [172.16.176.142]

TASK [doSSH : LOCAL blank temp file] *******************************************
ok: [172.16.176.142]

TASK [doSSH : REMOTE blank temp file] ******************************************
ok: [172.16.176.142 -> localhost]

TASK [doSSH : Write users to temp REMOTE file] *********************************
changed: [172.16.176.142] => (item={u'groups': u'ops', u'grp_ops': True, u'name': u'user1', u'key': u'user1 line one\nuser1 line two'})
changed: [172.16.176.142] => (item={u'groups': u'ops', u'grp_ops': True, u'name': u'user2', u'key': u'user two, line 1\nuser two, line 2\nuser two, line 3'})
changed: [172.16.176.142] => (item={u'groups': u'dev', u'name': u'devuser', u'key': u'ssh-rsa AAABBBCCC sshdevuser@nogroup'})

TASK [doSSH : Write users to LOCAL temp file] **********************************
changed: [172.16.176.142 -> localhost] => (item={u'groups': u'ops', u'grp_ops': True, u'name': u'user1', u'key': u'user1 line one\nuser1 line two'})
changed: [172.16.176.142 -> localhost] => (item={u'groups': u'ops', u'grp_ops': True, u'name': u'user2', u'key': u'user two, line 1\nuser two, line 2\nuser two, line 3'})
changed: [172.16.176.142 -> localhost] => (item={u'groups': u'dev', u'name': u'devuser', u'key': u'ssh-rsa AAABBBCCC sshdevuser@nogroup'})

PLAY RECAP *********************************************************************
172.16.176.142             : ok=5    changed=2    unreachable=0    failed=0
