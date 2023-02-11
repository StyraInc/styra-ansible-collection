# Ansible Collection - styra.opa


## Overview
A **Policy Decision Point (PDP)** is a piece of software that when asked for a policy decision will evaluate a policy against the query and return a response.  It is designed to **decouple policy** from an underlying piece of software so that policies (which are so crucial to security, compliance, and operations) **can be managed separately from the software that enforce those policies**.

[Open Policy Agent (OPA)](https://www.openpolicyagent.org/) is a **policy decision point created by Styra** and owned by the Cloud Native Computing Foundation.  It has attained the highest maturity level of CNCF projects ("graduated"), which is the same as maturity level as Kubernetes, Prometheus, Envoy, etc.


OPA is a lightweight PDP and is designed to distribute the load of policy decision making as close to the edge of a software application or infrastructure as possible.  By running OPA near the edge, policy decisions achieve high performance and high availability by a shared-fate architecture.

Styra also provides a **commercial-grade management plane for OPA** that implements a **single pane of glass** for controlling the many instances of OPA that are often required for a full policy solution.  This product called [Styra DAS](https://www.styra.com/styra-das/) provides
* full **policy lifecycle management** (authoring, testing, deploying, monitoring, and logging)
* **enterprise governance** (global policies that span many teams)
* **specialized support for common OPA use cases** (kubernetes, terraform IaC, envoy-based service meshes, kong gateway)

This ansible collection provides functionality to install OPA as a standlone PDP.  (Often OPA is run as a sidecar as part of a Pod instead of as a standalone service.  Such an installation is out of scope as it requires modifying for example k8s Deployments that make up the application.) There are 3 possible configuration options supported by this ansible collection:

* **barebones**:  No usage of any of the [OPA management configuration](https://www.openpolicyagent.org/docs/latest/management-introduction/) settings.  You are responsible for pushing policies and data into OPA so that it can make decisions.  Not recommended for production use.
* **bundle**: Includes [bundle configuration](https://www.openpolicyagent.org/docs/latest/management-bundles/) so that policy/data bundles are served from an HTTP server.
* **das_discovery**: Includes [discovery configuration](https://www.openpolicyagent.org/docs/latest/management-discovery/), which downloads bundle/status/decision-log configuration and then applies it.  Specifically tailored to [Styra DAS](https://www.styra.com/styra-das/).



## Ansible playbook usage

Below are sample playbooks for each of the configuration options supported by this collection

```yaml
# Run OPA with minimal configuration
- name: Barebones -- Run OPA as a PDP on k8s so that you can push policies and data into it
  hosts: localhost
  connection: local
  gather_facts: false
  roles:
    - name: k8s_pdp
      vars:
        config_mode: barebones
```

```yaml
# Run OPA and have it download policy from the OPA Playground
- name: Bundle -- Run OPA as a PDP on k8s and have it download a bundle of policy and data
  hosts: localhost
  connection: local
  gather_facts: false
  roles:
    - name: k8s_pdp
      vars:
        config_mode: bundle
        opa_bundle_url: https://play.openpolicyagent.org
        opa_bundle_resource: bundles/Ef9aP7LCso
        opa_bundle_polling: 45
```


```yaml
# Run OPA and use Styra DAS to provide policies, track status, and record decisions
- name: DAS-Discovery -- Run OPA as a PDP on k8s and connect it to Styra DAS for policy lifecycle management
  hosts: localhost
  connection: local
  gather_facts: false
  roles:
    - name: k8s_pdp
      vars:
        config_mode: das_discovery
        # DAS is multi-tenant, so set your tenant name
        das_tenant: test
        # A single DAS tenant has multiple policy projects (aka "systems"), so specify which one to use
        das_system_id: cad1b43326b845678121f20df61086e1
        # DAS requires an API token for OPA to request bundles and upload status and decision logs,
        #  so set the token by reading it from a file.
        das_token: "{{ lookup('file', 'tmp/token') }}"
```


## Walkthrough

This walkthrough presumes you are using minikube, but the same will work for any k8s cluster.

```
minikube start
```

Start with an ansible playbook (called `opapdp.yml`).
```
- name: Bundle -- Run OPA as a PDP on k8s and have it download a bundle of policy and data
  hosts: localhost
  connection: local
  gather_facts: false
  roles:
    - name: k8s_pdp
      vars:
        config_mode: bundle
        opa_bundle_url: https://play.openpolicyagent.org
        opa_bundle_resource: bundles/Ef9aP7LCso
        opa_bundle_polling: 45
```

Run the playbook:
```
$ ansible-playbook opapdp.yml
```

To interact with OPA on minikube, first forward the proper port.
```
$ kubectl port-forward service/opa 8181:8181
```

Then in a different terminal ask OPA to make a policy decision for a given input:
```
$ curl -X POST http://localhost:8181/v1/data/play/hello -d '{"input": {"message": "world"}}'
{"result":true}
```

Clean up after yourself by removing OPA from the k8s cluster by adding `state: absent` to your playbook.
```
- name: Bundle -- Run OPA as a PDP on k8s and have it download a bundle of policy and data
  hosts: localhost
  connection: local
  gather_facts: false
  roles:
    - name: k8s_pdp
      vars:
        config_mode: bundle
        opa_bundle_url: https://play.openpolicyagent.org
        opa_bundle_resource: bundles/Ef9aP7LCso
        opa_bundle_polling: 45
        state: absent
```

```
$ ansible-playbook opapdp.yml
```

## Variables
The variables provided by the PDP k8s server are detailed below.  Some variables are always useful,
while others are useful only for certain `config_mode`s.


### Global Variables

The following variables apply to all `config_mode`s.

|Variable|Description|Possible Values|Default|
|---|---|---|---|
|config_mode|How to configure OPA for policy, status, logs| barebones, bundle, das_discovery| bundle |
|state|Whether to install or uninstall OPA|present, absent| present |
|opa_name|The name for opa's k8s Deployment, Service, etc. | k8s string | opa |
|opa_namespace|The k8s namespace for OPA (will create if does not exist)| k8s string | default |
|opa_distribution|The name of the k8s image for OPA | string | openpolicyagent/opa |
|opa_version|The version of the k8s image for OPA| string | 0.48.0-rootless |
|debug|Whether to record the k8s manifests for inspection|true, false| false |
|debug_output_dir|The directory in which to store k8s manifests.  Must exist|string | /tmp/opa|


### Bundle Variables

The following variables apply when the config_mode is `bundle`.

|Variable|Description|Possible Values|Default|
|---|---|---|---|
| opa_bundle_url | URL portion of the bundle configuration, e.g. https://play.openpolicyagent.org | string | *none* |
| opa_bundle_resource | The resource portion of bundle configuration, e.g. bundles/Ef9aP7LCso | string | *none* |
| opa_bundle_polling | The number of seconds between OPA polls of the bundle server | number of seconds | 45 |

### DAS Variables

The following varaibles apply when the config_mode is `das_discovery`.

|Variable|Description|Possible Values|Default|
|---|---|---|---|
| das_system_id | The ID for the DAS system that this OPA is connecting to | string | *none* |
| das_system_type | The specific type of the DAS system to which this OPA is participating | string | custom |
| das_tenant | The tenant name in which the system_id resides and the API token exists | string | *none* |
| das_system_token | The API token for OPA to connect to DAS.  Must be granted discovery, bundles, status, decision logs | string | *none* |

## Requirements

This collection requires

* Python 3.9
* Ansible-core 2.14



