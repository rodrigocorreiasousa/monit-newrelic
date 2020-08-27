Ansible K8S New Relic Role
==========================

Ansible role to set up the monitoring stack for k8s clusters using New Relic and the azure logging agent.

Playbook installs and manages services. Currently supported:
  - New Relic Kube State Metrics
  - New Relic Events
  - New Relic Prometheus
  - New Relic Infrastructure
  - New Relic Logs
  - Azure Logging Agent

The role includes extensive configuration options check default/main.yml

Requirements
------------

- Ansible version 2.7 (tested with ansible 2.7.7)


Role Variables Default
----------------------

```yaml
---
# CONFIGURATION DEFAULT
namespace_newrelic: "nri-monitoring"
nri_registry_url: "newrelic"

#AGENT AZURE LOG
configure_azm_ms_agent: false
azure_log_stdout: "false"
azure_log_stderr: "false"

#AGENT INFRASTRUCTURE
nri_image_infra: "infrastructure-k8s:1.22.0"
#Configuration of Agent Infrastructure
#infrastructure integration with services like cadvisor, etcd, kube state metrics, api, controller, scheduler.
nri_integration_infra:
- {"#- name:": "KUBE_STATE_METRICS_POD_LABEL", "  #value:": "<YOUR_LABEL>"}
#infrastructure configuration parameters
nri_config_infra:
- {"integration_name:": "com.newrelic.kubernetes", "instances:": "", "  - name:": "nri-kubernetes", "    command:": "metrics"}
#parameters of loglevel
nri_nria_verbose_infra: "0"
#limits e requests
nri_limits_memory_infra: "300M"
nri_requests_cpu_infra: "100m"
nri_requests_memory_infra: "150M"

#AGENT KUBE EVENTS
nri_image_kube_events: "nri-kube-events:1.1.0"
#Configuration of Agent Kube Events
#parameter to add variables in kube events
nri_variables_events:
#kube events configuration parameters
nri_header_events:
- {"- name:": "newRelicInfra"}
nri_config_events:
  agentEndpoint: http://localhost:8001/v1/data
  clusterName: "{{ cluster_name }}"
  agentHTTPTimeout: 30s
#parameters of loglevel 
nri_loglevel_kube_events: "debug" 
#limits e requests 
nri_limits_cpu_kube_events: "500m"
nri_limits_memory_kube_events: "128Mi"
nri_requests_cpu_kube_events: "100m"
nri_requests_memory_kube_events: "128Mi"

#AGENT KUBE INFRA AGENT
nri_image_forwarder_events: "k8s-events-forwarder:1.11.24"
#limits e requests 
nri_limits_cpu_kube_infra: "500m"
nri_limits_memory_kube_infra: "128Mi"
nri_requests_cpu_kube_infra: "100m"
nri_requests_memory_kube_infra: "128Mi"

#AGENT KUBE STATE METRICS
nri_registry_url_kube_state: "quay.io"
nri_image_kube_state: "coreos/kube-state-metrics:v1.7.2"
nri_prom_scrape_kube_state: "true"

#configuration of Agent Metadata Injection Job 
nri_image_cert_manager: "k8s-webhook-cert-manager:1.2.1"
#configuration of Agent Metadata Injection 
nri_image_metadata_injection: "k8s-metadata-injection:1.2.0"
nri_limits_memory_meta_injection: "80M"
nri_requests_cpu_meta_injection: "100m"
nri_requests_memory_meta_injection: "30M"

#AGENT NEW RELIC LOG 
nri_image_log: "newrelic-fluentbit-output:1.3.0"
#Configuration of Agent New Relic Log
#service fluentbit
nri_service_fluent_bit_log:
- {"Flush": "1", "Log_Level": "info", "Daemon": "off", "HTTP_Server": "On", "HTTP_Listen": "0.0.0.0", "HTTP_Port": "2020"}
#Input for the logs of Kubernetes
nri_input_kubernetes_log:
- {"Name": "tail", "Tag": "kube.*", "Path": "/var/log/containers/*.log", "Parser": "docker", "DB": "/var/log/flb_kube.db", "Mem_Buf_Limit": "7MB", "Skip_Long_Lines": "On", "Refresh_Interval": "10"}
#Filters for the logs of Kubernetes
nri_filter_type_kubernetes_log:
- {"Name": "kubernetes", "Match": "kube.*", "Kube_URL": "https://kubernetes.default.svc.cluster.local:443", "Merge_Log": "Off"}
nri_filter1_log:
- {"Name": "grep", "Match": "kube.*", "Exclude stream": "stdout"}
nri_filter2_log:
#output fluentbit
nri_endpoint_newrelic_log: "https://log-api.newrelic.com/log/v1"
#limits e requests 
nri_limits_cpu_log: "500m"
nri_limits_memory_log: "128Mi"
nri_requests_cpu_log: "250m"
nri_requests_memory_log: "64Mi"

# AGENT PROMETHEUS NEW RELIC # ANALISE # CONFIG MAP add.
nri_image_prom: "nri-prometheus:1.3.0"
#Configuration of Agent Prometheus New Relic
#global
nri_scrape_duration_prom: "30s"
nri_scrape_timeout_prom: "5s"
nri_telemetry_expiration_age_prom: "5m"
nri_telemetry_expiration_interv_prom: "5m"
nri_verbose_prom: "false"
nri_insecure_skip_verify_prom: "false"
scrape_enabled_label_prom: "prometheus.io/scrape"
require_scrape_enabled_label_prom: "true"
nri_rules_general_prom:
  transformations:
    ignore_metrics:
      - prefixes:
        - nginx_ingress_controller_ingress_upstream_latency_seconds
        - certmanager_http_acme_client_request_duration_seconds
```

[DOCS: New Relic Kube State Metrics](https://docs.newrelic.com/docs/infrastructure?toc=true)

[DOCS: New Relic Events](https://docs.newrelic.com/docs/insights/event-data-sources/custom-events)

[DOCS: New Relic Prometheus](https://docs.newrelic.com/docs/integrations/prometheus-integrations)

[DOCS: Prometheus](https://prometheus.io/docs/prometheus/latest/configuration/configuration)

[DOCS: New Relic Infrastructure](https://docs.newrelic.com/docs/infrastructure?toc=true)

[DOCS: New Relic Logs](https://docs.newrelic.com/docs/logs?toc=true)

[DOCS: Fluent Bit](https://docs.fluentbit.io/manual)

Dependencies
------------

None.

Install
-------

Add the following on your ``requirements.yml``:
```yaml
- name: stone-payments.k8s-newrelic
  src: git@github.com:stone-payments/ansible-k8s-newrelic.git
  scm: git
  version: "v1.0.0" # see the git tags for available versions
```

Example Playbook
----------------

The following parameters are expected:
- `kubeconfig_path`: The path to a kubeconfig file that contains the credentials to login to the cluster.
- `cluster_name`: This parameter is to add the name of the cluster so you can identify it in New Relic.
- `license_key`: This parameter is for adding the account license.
- `configure_azm_ms_agent`: This parameter is for the azure agent does not capture application logs.

With all that defined, just call it in ansible:

```yaml
- hosts: localhost
  tasks:
    - import_role: 
        name: stone-payments.k8s-newrelic
      vars:
        kubeconfig_path: "..."
        cluster_name: "..."
        license_key: "..."
        nri_filter1_log:
        - {"Name": "grep", "Match": "kube.*", "Exclude stream": "stdout"}
        configure_azm_ms_agent: "true"
```# monit-newrelic
