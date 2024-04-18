
# Overview

To develop an ansible collection, consult [Redhat's Ansible Certification Workflow Guide](https://connect.redhat.com/sites/default/files/2022-08/Ansible%20Certification%20Workflow%20Guide%202022.pdf)

This collection, `styra.opa`, is deployed to:
* [Ansible Galaxy](https://galaxy.ansible.com/) under [styra/opa](https://galaxy.ansible.com/styra/opa).  Galaxy is an open-source repository for ansible collections.
* [Automation Hub](https://console.redhat.com/ansible/automation-hub) under [styra/opa](https://console.redhat.com/ansible/automation-hub/repo/published/styra/opa/).  Automation Hub is a supported, proprietary variant of Ansible Galaxy.

You must publish each version to both Galaxy and Automation Hub


## Running locally

Install Docker/minikube and then start up k8s.

```
minikube start
```

Setup a Python virtual environment
```
python3 -m pip install venv env
source env/bin/activate
```

Install python dependencies

```
python3 -m pip install -r ansible_collections/styra/opa/requirements.txt
```

Install ansible dependencies
```
ansible-galaxy install kubernetes.core --token TOKEN
```

You may procure a token by logging in to galaxy.ansible.com/USERNAME/preferences and asking for the `API key`

Choose one of the playbooks in README.md and put into a file `play.yml`.  Then run the ansible playbook.

```
ansible-playbook play.yml
```

To undo the ansible playbook, use the `state: absent` modifier in the playbook and rerun it as shown above, e.g.

```
- name: Barebones -- Run OPA as a PDP on k8s so that you can push policies and data into it
  hosts: localhost
  connection: local
  gather_facts: false
  roles:
    - name: styra.opa.k8s_pdp
      vars:
        config_mode: barebones
      state: absent
```

## Publishing the collection to Ansible Galaxy

To upload an ansible collection to galaxy, you must

```
$ vi ansible_collections/styra/opa/galaxy.yml
$ <bump the version>
```

Then you must build the collection and upload it:

```
$ cd ansible_collections/styra/opa
$ ansible-galaxy collection build .
$ ansible-galaxy collection publish --token TOKEN styra-opa-MAJOR.MINOR.PATCH.tar.gz
```

You may procure a token by logging in to galaxy.ansible.com/USERNAME/preferences and asking for the `API key`

You can check that it has been uploaded by visiting [https://galaxy.ansible.com/styra/opa](https://galaxy.ansible.com/styra/opa).


## Publishing the collection to Automation Hub

To publish to the Automation Hub, you can create a file ansible.cfg file.  It took some trial and error to figure out what this needed to look like:

```
[galaxy]
server_list = automation_hub

[galaxy_server.automation_hub]
url=https://console.redhat.com/api/automation-hub/content/inbound-styra/
auth_url=https://sso.redhat.com/auth/realms/redhat-external/protocol/openid-connect/token
token=TOKEN
```

Then you publish the same as for the Ansible Galaxy but provide the ANSIBLE_CONFIG env variable.
```
$ cd ansible_collections/styra/opa
$ ansible-galaxy collection build .
$ ANSIBLE_CONFIG=../../../ansible.cfg ansible-galaxy collection publish styra-opa-MAJOR.MINOR.PATCH.tar.gz
```
You may procure a token by logging in to the [Ansible Automation Platform](https://console.redhat.com/ansible/automation-hub) and going to [Automation Hub/Connect to Hub](https://console.redhat.com/ansible/automation-hub/token).

Someone at Redhat will manually review the upload, and will release it.  Ideally, we should email `ansiblepartners@redhat.com` to let them know there is a new upload to approve, and so they can contact us easily with questions and status updates.

Once it is released, you may view it at [https://console.redhat.com/ansible/automation-hub/repo/published/styra/opa/](https://console.redhat.com/ansible/automation-hub/repo/published/styra/opa/).


## Additional resources

Example collections
* [Kong](https://github.com/Kong/kong-ansible-collection) Example of an installer for k8s
* [Arista](https://github.com/aristanetworks/ansible-avd) Example of same directory structure and test suite

