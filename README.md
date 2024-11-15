
![](./img/Ansible-and-Openshift.png? ':size=50%')

# Automated OpenShift Cluster Health Report

Automation has become a fundamental way to enhance efficiency, reduce human errors, and accelerate processes within IT infrastructure today. By automating repetitive tasks and complex workflows, operations can run smoothly without manual intervention, allowing IT teams to focus on more strategic, value-generating work. Furthermore, automation provides scalability and consistency essential for keeping up with rapidly growing digital demands, ensuring the security and continuity of systems.

For OpenShift environments located across different remote locations, the automation solution I developed offers a centralized way to monitor and report on the health status of all clusters regularly. This automation immediately sends notifications via Microsoft Teams and email when an issue is detected, enabling infra and DevOps teams to respond quickly. Consequently, the reliability and performance of OpenShift infrastructure across multiple locations are monitored and managed more effectively, delivering greater resilience and efficiency.


> [!NOTE|style:flat]
> This playbook tested with 4.11.x, 4.12.x, 4.13.x and 4.14.38 Openshift version.
> Some modules may work for UPI clusters.

### Features

* Generates a stand-alone HTML file for OpenShift cluster health
* Requires oc, jq, and python-jmespath packages on Ansible nodes
* Light-weight and provides table filtering functionality, allowing users to filter and view specific data within tables
* OC commands are run directly on the Ansible node
* Modular design, making it easy to add new components
* Available Modules:
	* get-token: Retrieves the access token for API requests
	* get-nodes: Collects node information, including health and resource usage
	* get-co: Gathers ClusterOperator status for component health
	* get-resource: Checks available and used resources across nodes
	* get-node-conditions: Reports on node conditions and possible memory, cpu, kubelet services issues
	* get-pods-distribution: Displays distribution of pods across nodes
	* get-etcd-stats: Collects ETCD pods status
	* get-update-status: Checks for cluster update progress and status
	* get-deployments: Gathers deployment statuses in namespaces. Single running pod are highlighted
	* get-ssl: Validates SSL certificates and expiration dates
	* get-alert: Retrieves active alerts within the cluster
	* get-pod-health: Checks health and status of individual pods
	* get-overcommitted: Get overcommitted  node status
	* get-pv: Monitors Persistent Volume status and availability
	* get-pod-status: Collects overall pod status across the cluster
	* get-pod-usage: Checks resource usage metrics of pods
	* get-ns: Lists namespaces and their statuses
	* get-route: Route lists
	* get-total-resource: Summary of total cluster resources
	* get-operators: Cluster Operator statuses and health
	* get-users-group: Lists users and groups
	* get-machine: Gathers machine information
	* get-daemonset: Checks DaemonSet statuses
	* get-apps: Retrieves application statuses and configurations
	* send-mail: Sends the generated report via email.

### How to use
Install packages

```bash
yum install python3-jmespath jq
```

Install collections   community.general, ansible.posix, ansible.utils


```bash
ansible-galaxy install -r collections/requirements.yml
```

Clone or copy this repository on the ansible node

```bash
git clone https://github.com/emrahuludag/openshift-report.git
```

Edit variables;

```bash
vi ./vars/main.yaml 
```

```bash
site_name: "TRIST"
cluster_name: "TR-DEV"

#
# Authorized user
#
ocp_user: "user"
ocp_pass: "pass"
ocp_url: "https://api.exampledomain.local:6443"

#
# Mail Relay Configuration
#
mail_host: "1.1.1.1"
mail_port: "25"
mail_sender: "cluster-report@exampledomain.local (Cluster Overview Automation)"
mail_to: "emrahuludag@gmail.com, devops@exampledomain.local"

#
# Console and App UI Certificate Checks
#
web_console: "console-openshift-console.apps.exampledomain.local"
app_ui: "console-openshift-console.apps.exampledomain.local"

```

And run the ocp4-report.yml playbook.

```bash
ansible-playbook ocp4-report.yml
```


### Schedule Report


Create script daily.sh

```bash
#!/bin/sh
/usr/bin/ansible-playbook /openshift-report/ocp4_report.yml
```

Add cron job 
Script will run 07:00 AM 

```bash
0 7 * * * /openshift-report/daily.sh
```


### Sample Report

Right click to save [OpenShift Full Sample Report](https://github.com/emrahuludag/sysknow/raw/main/docs/openshift/img/openshift-sample-report.html)

![](./img/ocp-report-sample01.png? ':size=80%')
![](./img/ocp-report-sample02.png? ':size=80%')
![](./img/ocp-report-sample03.png? ':size=80%')
![](./img/ocp-report-sample04.png? ':size=80%')
![](./img/ocp-report-sample05.png? ':size=80%')

![](./img/ocp-report-sample06.png? ':size=80%')

![](./img/ocp-report-sample07.png? ':size=20%')

# For Monitoring
# Health Check OpenShift Cluster with AWX

This playbook connects to OpenShift (IPI) clusters via AWX, monitoring the health of nodes and cluster operators. If any issues are detected, it promptly sends alerts through both email and Microsoft Teams, ensuring rapid awareness and response to cluster health concerns


> [!NOTE|style:flat]
> This playbook tested with 4.11.x, 4.12.x, 4.13.x and 4.14.38 Openshift version.
> Some modules may work for UPI clusters.

### Features

* Generates a stand-alone HTML file for OpenShift cluster health
* Requires oc, jq, and python-jmespath packages on Ansible nodes
* Light-weight and provides table filtering functionality, allowing users to filter and view specific data within tables
* OC commands are run directly on the Ansible node
* Modular design, making it easy to add new components
* Available Modules:
	* get-token: Retrieves the access token for API requests
	* get-nodes: Collects node information, including health and resource usage
	* get-co: Gathers ClusterOperator status for component health
	* get-resource: Checks available and used resources across nodes
	* get-node-conditions: Reports on node conditions and possible memory, cpu, kubelet services issues
	* get-etcd-stats: Collects ETCD pods status
	* get-alert: Retrieves active alerts within the cluster
	* get-pod-health: Checks health and status of individual pods
	* get-total-resource: Summary of total cluster resources
	* send-mail: Sends the generated report via email and microsoft teams.

### How to use


Install packages on Ansible Nodes

```bash
yum install python3-jmespath jq
```

Install collections   community.general, ansible.posix, ansible.utils

```bash
ansible-galaxy install -r collections/requirements.yml
```

Prepare a Teams channel to receive Webhooks
I have created a new channel called “Cluster Alerts”. You can configure webhooks for this channel by easily going into the “Connectors” option. Then, create api url and configure playbook.

![](./img/awx-ocp407.png? ':size=20%')

![](./img/awx-ocp408.png? ':size=20%')

![](./img/awx-ocp409.png? ':size=20%')


Playbook > roles/send-mail-monitoring/tasks/main.yml

```bash
- name: Send a notification to Teams Channel
  uri:
    url: "https://abc.webhook.office.com/webhookb2a44b3/IncomingWebhook/7e2c20265d66cFQqYrrvQCNGEE6Ok1"
    method: POST
    body_format: json
    headers:
      Content-Type: "application/json"
    body:
      title: "{{site_name}}, {{cluster_name}}"
      text: "{{ teams_text }}"
      sections:
        - facts:
          - name : "Cluster URL"
            value: "{{ web_console }}"
          - name: "Date"
            value: "{{ dash_date }}"
    status_code: 200
  when: (node_critical_flag  == true) or (co_warning_flag == true)
```

Create AWX Project and sync git repo.

Git repo

```bash
https://github.com/emrahuludag/openshift-report.git
```

![](./img/awx-ocp401.png ':size=50%')


Create credentials for openshift user in awx 

![](./img/awx-ocp402.png? ':size=50%')


Create AWX Template for openshift-report. Then, select ocp4-report_monitor_awx.yml playbook. Then edit variables;

![](./img/awx-ocp403.png? ':size=50%')


```bash

site_name: "TRIST"
cluster_name: "TR-DEV"
mail_host: "1.1.1.1"
mail_port: "25"
mail_sender: "cluster-report@exampledomain.local (Cluster Overview Automation)"
mail_to: "emrahuludag@gmail.com, devops@exampledomain.local"
web_console: "console-openshift-console.apps.exampledomain.local"
app_ui: "console-openshift-console.apps.exampledomain.local"

```

### Schedule awx templates or awx workflow

If you have multiple environments, you can create a workflow to add each of them in sequence. Then, you can schedule the template or workflow you've created to run automatically

![](./img/awx-ocp404.png? ':size=50%')


### Sample Report and Alerts

![](./img/awx-ocp405.png? ':size=50%')

![](./img/awx-ocp406.png? ':size=50%')
