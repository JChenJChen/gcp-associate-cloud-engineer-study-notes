## GCP Hierarchy -- Billing & Permissions

- organization: top level node
- folders: mainly used for organization, often mimic company structure
- permissions are inheritable:
  - child resources inherit permissions from parent, which CANNOT be removed by child or another resource at same or below level
  - child resource CAN ADD permissions in addition to inherited permissions
- role:
  - collection of permissions that define what tasks can be performed on what resources
  - can be assigned to users to perform tasks
  - 3 types:
    - basic
      - owner: manage roles & permissions
        - invite members
        - remove members
        - delete projects
        - plus editor and viewer perms
      - editor: can change state
        - deploy apps
        - modify code
        - configure services
        - plus viewer perms
      - viewer˜
        - read-only
      - billing administrator
    - IAM predefined -- i.e. on a resource type
    - custom
- Owning a project & being able to control its resources is completely separate from being able to control the billing account tied to that project → admin account that created 2 projects can see 3 projects but not access the 1 project that’s created by user.
- By default, users are allowed to create project and billing accounts, so should be tuned.
  - assign users to roles for fine-grained permissions
- relationships:
  - org->billing: many
  - billing->project: many
  - project->billing: single
## Ways to interact with GCP resources and services

1. Web: GCP console
2. CLI: Cloud shell and cloud SDK
3. Mobile: Cloud console App
4. REST API: for custom applications

### client libraries to control GCP resources with code

- cloud client libraries
- google API client libraries
  - open source, generated
  - multi-language support

## GCP Structure & Design
- Physical Infrastructure
    - Ppl who work in data center do no have access to services, and ppl who work in services do not have access to data center
- Network ingress & egress: by default, data spends as much time inside the private network as possible. But a “normal” egress mode is available to reduce costs
- Persistent disk
    - VM-attached disks
    - Block storage (even divided blocks of data with only address & no metadata) – as opposed to Cloud storage → Object storage (data, expandable metadata, global unique identifier)
- Cloud Filestore: Managed NFS that’s more flexible to connect to than persistent disk
- Firestore: strongly consistent serverless document database. Cool feature: web socket connections for updates
- Cloud datastore: usage-based horizontally scalable document database
- Dataproc = managed spark + hadoop
- Dataflow: newer & better than dataproc. Both stream & batch data processing
- Cloud Hardware Security Module Service (HSM): manage encryption keys & certificates


## GCP Projects & Naming

| Project Attribute | Unique?            | Who sets it?    | Mutable?  |
|-------------------|--------------------|-----------------|-----------|
| Project ID        | Globally unique    | user selected   | immutable |
| Project Name      | Need not be unique | user selected   | mutable   |
| Project Number    | Globally unique    | Google assigned | immutable |

## GCloud Shell
- Nodemon?

## Dataflow
- Moving → Network
- Processing → Compute
- Remembering → Storage

## gsutil
- gsutil ls gs://<bucket_name>/**
    - /** shows subdirectory contents in flat results
- gsutil mb -l northamerica-northeast1 gs://storage-lab-cli
- gsutil label get gs://storage-lab-console-jc-20220131 >bucket_labels.txt
```
{
  "function": "console-test"
}
```
- gsutil label get gs://storage-lab-console-jc-20220131 >bucket_labels.json
- gsutil label set bucket_labels.json gs://storage-lab-cli-jc-2022-0131
- gsutil label ch -l "extralabel:extravalue" gs://storage-lab-cli-jc-2022-0131
- gsutil versioning get gs://storage-lab-cli-jc-2022-0131
- gsutil versioning set on gs://storage-lab-cli-jc-2022-0131
- gsutil ls gs://storage-lab-cli-jc-2022-0131/
- gsutil cp bucket_labels.json gs://storage-lab-cli-jc-2022-0131/
- gsutil ls -a gs://storage-lab-cli-jc-2022-0131/ → -a for archive, shows object versioning
- gsutil cp gs://storage-labl-console-jc-20220131/** gs://storage-lab-cli-jc-2022-0131
    - /** copies all objects regardless of folder hierarchy, BUT different from -r recursive flag bc cmd will copy all files to the destination without hierarchy
    - Share settings are not copied
- gsutil acl ch -u AllUser:R gs://<bucket>/<object>
    - acl = access control list
    - ch = change
    - -u = user
    - R = read
- gsutil rm
- gsutil mb -l <location> <gs://bucket_name>

## VM

- gcloud config get-value project
- gcloud config list project
- gcloud config list --all
- gcloud config list compute/zone
- gcloud compute instances list
  - cmd initially gets 403 bc compute API isn't enabled -- but actually running cmd gets PERMISSION DENIED, no 403
- gcloud services list
    - gcloud services list --enabled
    - gcloud services list --available
    - gcloud services list --available | grep compute
    - gcloud services enable compute.googleapis.com
        - enabling an API may enable other dependent or related APIs as well (i.e. oslogin)
        - also created cloud services & compute@developer service accounts
    - gcloud compute instances create myvm
  - --help gets full help docs, -h gets simpler help docs

## Rundown on gcloud
- `gsutil` & `bq` share same configuration set via `gcloud config`
  - `gsutil` -> gcloud storage
  - `bq` -> gcloud bigquery
- console >> gcloud CLI >> gcloud REST API
```
gcloud <global_flags> <service/product> <group/area> <command> <flag> <parameters>
```
> gcloud cmds always specifies service/product before cmd
### global flags
- --help
- -h
- --project <projectID>
- --acount <Account>
- --filter (not always available, but usually better than grep)
- --format [JSON|YAML|CSV|etc]
  - can pipe JSON to jq
### config properties
basically env vars but can be overridden within cmds w/ flags
- i.e.: account, project, region, zone. Ex:
  - `core/account`|`account` -> --account
  - `core/project`|`project` -> --project
  - `compute/region` -> --region
  - `compute/zone` -> --zone
- gcloud config set <property> <value>
- gcloud config get-value <property>
- gcloud config unset <property>

### Configurations: groups of settings

- groups of setttings to toggle between, useful with multiple projects
```
gcloud init
## SQL
```
- gcloud sql operations list --instance [INSTANCE_NAME] --project [PROJECT_NAME]

### Configurations Analogy

|Action | Directory | Configuration|
|------|-----------|----------------|
|Make New | mkdir newdir | gcloud config configurations create newconfig |
|Switch To | cd newdir | gcloud config configurations activate newconfig |
|List Contents | ls | gcloud config list |
|List Non-active | ls ~/newdir | gcloud --configuration=newconfig config list OR gcloud config configurations describe newconfig|

## App Engine Standard Environment

- supports specific versions of Java, Python, PHP, & Go
- sandbox constraints:
  - no writing to local files
  - all requests time out at 60s
  - limits on 3rd party software


## VPC

- each subnet must have a primary range
- optional: up to 5 secondary ranges for an alias IP
- subnets must be RFC1918 addresses
- all IPs must be unique, but don't need to be contiguous
- firewall rules: global, apply by instance-level tags or SAs
  - default behavior: restrictive inbound, permissive outbound

### creating VPC with gcloud

- `gcloud compute networks create ace-exam-vpc1 --subnet-mode=[auto/custom]`
- `gcloud beta compute networks subnets create ace-exam-vpc-subnet1 --network=aceexam-vpc1 --region=us-west2 --range=10.10.0.0/16 --enable-private-ip-googleaccess --enable-flow-logs`

### creating VPC with gcloud

- first need to grant permissions either...:
  - to org member: `gcloud organizations add-iam-policy-binding [ORG_ID] --member='user:[EMAIL_ADDRESS]' --role="roles/compute.xpnAdmin"`
  - or to folder: `gcloud beta resource-manager folders add-iam-policy-binding [FOLDER_ID] --member='user:[EMAIL_ADDRESS]' --role="roles/compute.xpnAdmin"`
    - can find folder ID with: `gcloud beta resource-manager folders list --organization=[ORG_ID]`
- `gcloud (beta, if sharing at folder level. Sans if at org level) compute shared-vpc enable [HOST_PROJECT_ID]`
- associate projects to shared VPC:`gcloud (beta, if sharing at folder level. Sans if at org) compute shared-vpc associated-projects add [SERVICE_PROJECT_ID] --host-project [HOST_PROJECT_ID]`
- VPC peering can be used for interproject traffic when an organization does not exist. This peering will allow private traffic to flow between the two VPCs:
  -   ```
      gcloud compute networks peerings create peer-ace-exam-1 \
      --network ace-exam-network-A \
      --peer-project ace-exam-project-B \
      --peer-network ace-exam-network-B \
      --auto-create-routes
      ```
  -   And then vice versa: ```
      gcloud compute networks peerings create peer-ace-exam-1 \
      --network ace-exam-network-B \
      --peer-project ace-exam-project-A \
      --peer-network ace-exam-network-A \
      --auto-create-routes
      ```

### Deploying Compute Engine with Custom Network

- `gcloud compute instances create [INSTANCE_NAME] --subnet [SUBNET_NAME] --zone [ZONE_NAME]`


## Firewalls

- `gcloud compute firewall-rules create ace-exam-fwr2 –-network ace-exam-vpc1 –-allow tcp:20000-25000`

## VPNs

- gcloud compute target-vpn-gateways
- gcloud compute forwarding-rule
- gcloud compute vpn-tunnels
  - ex: `gcloud compute vpn-tunnels create NAME --peer-address=PEER_ADDRESS --sharedsecret=SHARED_SECRET --target-vpn-gateway=TARGET_VPN_GATEWAY`

## Networking

- [7 layers of the OSI model](https://www.webopedia.com/definitions/7-layers-of-osi-model/)
- IP address:
  - `abc.def.ghi.jkl` (dotted quad)
  - each piece can be 0-255
  - CIDR block: group of IPs specified in `<IP>/xy` notation
    - turn IP into 32 bit binary number
    - `/xy` in CIDR notation locks the highest (leftmost) bits in IP address (0-32)
    - [CIRD conversion table](https://techlibrary.hpe.com/docs/otlink-wo/CIDR-Conversion-Table.html)
    - [subnetting & subnet masks](http://www.steves-internet-guide.com/subnetting-subnet-masks-explained/)
- common ports:
  - HTTP: 80
  - HTTPS: 443
  - SSH: tcp:22
- CIDR blocks:
  - /32 -> 255.255.255.255
  - /31 -> 2 IPs
  - /30 -> 4 IPs
  - /29 -> 8 IP address, 4 usable, 4 are reserved by Google (only bits 30, 31, 32 can change. 3 digit binary can have 8 different values)
  - /28 -> 255.255.255.0 -> 16 addresses
  - /27 -> 32 addresses
  - /26 -> 64 addresses
  - /20 ->

resources:
- subnetting & CIDR: https://www.youtube.com/watch?v=s_Ntt6eTn94
- subnets cheat sheet: https://www.freecodecamp.org/news/subnet-cheat-sheet-24-subnet-mask-30-26-27-29-and-other-ip-address-cidr-network-references/

### Networking in the Cloud: DNS, Load Balancing, and IP Addressing

5 types of GCP LBs:
1. HTTP(S): balances HTTP and HTTPS load
2. SSL Proxy: terminates SSL/TLS connections
3. TCP Proxy:  terminates TCP sessions
4. Network TCP/UDP: load balancing is based on IP protocol, address, and port
5. Internal TCP/UDP: balances TCP/UDP traffic on private networks hosting internal VMs

### DNS managed zones

- create managed zone: `gcloud beta dns managed-zones create ace-exam-zone1 --description= --dnsname=aceexamzone.com. (if private: --visibility=private --networks=default)`
- add an A record:
  1. `gcloud dns record-sets transaction start --zone=ace-exam-zone1`
  2. `gcloud dns record-sets transaction add 192.0.2.91 --name=aceexamzone.com. --ttl=300 --type=A --zone=ace-exam-zone1`
  3. `gcloud dns record-sets transaction execute --zone=ace-exam-zone1.`
- To create a CNAME record, we would use similar commands:
  1. `gcloud dns record-sets transaction start --zone=ace-exam-zone1`
  2. `gcloud dns record-sets transaction add server1.aceexamezone.com. --name=www2.aceexamzone.com. --ttl=300 --type=CNAME --zone=ace-exam-zone1`
  3. `gcloud dns record-sets transaction execute --zone=ace-exam-zone1`

### Load Balancers

- sidenote: [eli5 HTTP(S), TLS, TCP, etc](https://www.nagekar.com/2018/07/eli5-how-https-works.html)

Types:
- HTTPS: cross-regional LB for web app
- specific port #'s, TCP only
  - SSL traffic (not HTTP): global SSL proxy LB
  - non-SSL TCP traffic: global TCP proxy LB
- regional LB: UDP or any other port #
- internal LB: accepts traffic on GCP internal IP, LBs across compute VMs


- global vs regional
  - global: routes all traffic to single any cast IP, IPV6 termination
    - types:
      - HTTP(S)
      - SSL Proxy (terminates SSL/TLS connections) -- used for non-HTTPS traffic
      - TCP Proxy (terminates TCP sessions at LB then fwd's traffic to backend servers)
  - regional: if only require IPV4 temrination
    - types:
      - internal TCP/UDP (on private network hosting internal VMs)
      - network TCP/UDP (enables balancing based on IP protocol, address, port) -- used for SSL & TCP traffic not supported by SSL proxy & TCP proxy LBs, respectively
- external vs internal
  - internal: TCP/UDP LB is the only internal LB
  - external: HTTP(S), SSL Proxy, TCP Proxy, network TCP/UDP LBs
- traffic type
  - HTTP(S) traffic requires global external
  - TCP: external global or regional, or internal regional
  - UDP: external or internal regional

#### Creating an LB

TCP example, via console:
1. select LB type, external/internal, multi/single region, connection termination (offload TCP or SSL processing to LB)
2. configure backend: specify name, region, network, backends, health check (name, protocol, port, health criteria)
3. configure frontend: name, subnetwork, internal IP config (ephemeral), port to fwd traffic to backend

Network LB, via gcloud:
1. `gcloud compute forwarding-rules create ace-exam-lb --port=80 --target-pool ace-exam-pool`
2. `gcloud compute target-pools add-instances ace-exam-pool --instances ig1,ig2`

### Managing IP Addresses

- expanding CIDR blocks: ` gcloud compute networks subnets expand-ip-range ace-exam-subnet1 --prefix-length 16`
- reserving IP address   : gcloud beta compute addresses create ace-exam-reserved-static1 --region=us-west2
--network-tier=PREMIUM
## GCE
- pay by the second for CPU & RAM, min. 1 min
- pay per GB traffic network zone/region egress

## Cloud function
- Node.js, python, Go, Java
- pay for per 100ms RAM & CPU (min. 100ms)

## Transfer

Data Transfer Appliance
- rackable, high capacity storage server to physically ship data to GCS
- ingest only; not a way to avoid egress charges
- 100TB or 480TB versions
- 480TB/week is faster than a saturated 6Gbps link

Storage Transfer Service
- copies objects for you, no need to setup transfer machine
- destination is always GCS bucket
- source can be S3, HTTP(S) endpoint, or another GCS bucket
- one-time or scheduled recurring transfers
- free to use, but pay for actions

## Cloud Filestore
- accessible to GCE & GKE thru VPCs via NFSv3 protocol
- primary use case: lift & shift -- migrating application to cloud
- file serving is fully managed, backup isn't
- pay for provisioned TBs:
  - "standard" (slow), min 1TB
  - "premium" (fast), min 2.5TB
- But: use cloud storage if:
  - variable workloads
  - number of client connecting exceeds cloud filestore's limit
  - data to store exceed filestore's limit

## Cloud Storage

## External Networking

Google Domains
- private Whois records
- built-in DNS or custom nameservers
- supports DNSSEC
- email fwd'ing with auto setup of SPF and DKIM (for built-in DNS)

Cloud DNS
- DNS lookup is on port 53
- 100% uptime
- public & private managed zones
- low latency globally
- supports DNSSEC
- managed via UI, CLI, API
- fixed fee per managed zone to store & distribute DNS records
- variable usage fee for DNS lookups

## Static IP

- charge by reserve static IP addresses in projects and assign to resources
  - regional IPs: GCE instances and network LBs
  - global IPs: global LBs
    - HTTP(S), SSL proxy, TCP proxy

## Cloud Load Balancing (CLB)

## CDN


## cloudsql

- `gcloud sql backups create ––async ––instance [INSTANCE_NAME]`
- `gcloud sql instances patch [INSTANCE_NAME] –backup-start-time [HH:MM]`

## BQ
- `bq ––location=[LOCATION] query ––use_legacy_sql=false ––dry_run [SQL_QUERY]`

- `bq --location=US show -j gcpace-project:US.bquijob_119adae7_167c373d5c3`
## BigTable
- `echo instance = ace-exam-bigtable >> ~/.cbtrc`
- `cbt createtable ace-exam-bt-table`
- `cbt createfamily ace-exam-bt-table colfam1`
- `cbt set ace-exam-bt-table row1 colfam1:col1=ace-exam-value`
- `cbt read ace-exam-bt-table `

## GCS
- change storage class: `gsutil rewrite -s [STORAGE_CLASS] gs://[PATH_TO_OBJECT]`
  - STORAGE CLASS: multi_regional, regional, nearline , or coldline
- move bucket location: `gsutil mv gs://[SOURCE_BUCKET_NAME]/[SOURCE_OBJECT_NAME] gs://[DESTINATION_BUCKET_NAME]/[DESTINATION_OBJECT_NAME]`

## Import/Export Data

### CloudSQL

1. grant SA permissions to bucket: `gsutil acl ch -u [SERVICE_ACCOUNT_ADDRESS]:W gs://[BUCKET_NAME]`
2. create DB export: `gcloud sql export/import sql/csv [INSTANCE_NAME] gs://[BUCKET_NAME]/[FILE_NAME] --database=[DATABASE_NAME]`

### Datastore

- The Datastore export commands produces a metadata file and a folder with the data as an output
1. `gcloud datastore export --namespaces="(default)" gs://${BUCKET}`
2. `gcloud datastore import gs://${BUCKET}/[PATH]/[FILE].overall_export_metadata`

### BQ

- ` bq extract --destination_format [FORMAT] --compression [COMPRESSION_TYPE] --field_delimiter [DELIMITER] --print_header [BOOLEAN] [PROJECT_ID]:[DATASET].[TABLE] gs://[BUCKET]/[FILENAME]`
- `bq load --autodetect --source_format=[FORMAT] [DATASET].[TABLE] [PATH_TO_SOURCE]`

### Cloud BigTable

```
java -jar bigtable-beam-import-1.6.0-shaded.jar export \
 --runner=dataflow \
 --project=[PROJECT_ID] \
 --bigtableInstanceId=[INSTANCE_ID] \
 --bigtableTableId=[TABLE_ID] \
 --destinationPath=gs://[BUCKET_NAME]/[EXPORT_PATH] \
 --tempLocation=gs://[BUCKET_NAME]/[TEMP_PATH] \
 --maxNumWorkers=[10x_NUMBER_OF_NODES] \
 --zone=[DATAFLOW_JOB_ZONE]
 ```

```
java -jar bigtable-beam-import-1.6.0-shaded.jar import \
 --runner=dataflow \
 --project=[PROJECT_ID] \
 --bigtableInstanceId=[INSTANCE_ID] \
 --bigtableTableId=[TABLE_ID] \
 --sourcePattern='gs://[BUCKET_NAME]/[EXPORT_PATH]/part-*' \
 --tempLocation=gs://[BUCKET_NAME]/[TEMP_PATH] \
 --maxNumWorkers=[3x_NUMBER_OF_NODES] \
 --zone=[DATAFLOW_JOB_ZONE]
 ```

### Dataproc

- `gcloud dataproc clusters create cluster-bc3d ––zone us-west2-a`
- export dataproc cluster config: `gcloud beta dataproc clusters export [CLUSTER_NAME] --destination=[PATH_TO_EXPORT_FILE]`
- import source config file: `gcloud beta dataproc clusters import [SOURCE_FILE]`

### Pub/Sub

- create topic: `gcloud pubsub topics create [TOPIC_NAME]`
- create subscription: `gcloud pubsub subscriptions create --topic [TOPIC_NAME] [SUBSCRIPTION_NAME]`
- send data to topic: `cloud pubsub topics publish [TOPIC_NAME] --message [MESSAGE]`
- read msg from subscription: `gcloud pubsub subscriptions pull --auto-ack [SUBSCRIPTION_NAME]`

### GCP Marketplace (Cloud Launcher)

- launch form:
  - deployment name
  - zone
  - machine type (preconfigured)
  - admin email
  - optional: phpMyAdmin (for wordpress & PHP apps)
- networking:
  - VM network & subnet
  - firewall rules to allow HTTP(S) traffic
- `gcloud deployment-manager deployments create quickstart-deployment --config vm.yaml`
- `gloud deployment-manager deployments describe quickstart-deployment`

## IAM

- `gcloud iam roles describe`
- `gcloud projects get-iam-policy`
- `gcloud projects add-iam-policy-binding [RESOURCE-NAME] --member user:[USEREMAIL] --role [ROLE-ID]`
- `gcloud iam roles create [ROLE-ID] --project [PROJECT-ID] --title [ROLE-TITLE] --description [ROLE-DESCRIPTION] --permissions [PERMISSIONS-LIST] --stage [LAUNCH-STAGE]`
  - stage: alpha, beta, GA, disabled
- need to know how to:
  - work with scopes
    - `gcloud compute instances set-service-account [INSTANCE_NAME] [--service-account [SERVICE_ACCOUNT_EMAIL] | [--no-service-account] [--no-scopes | --scopes [SCOPES,...]]`
  - assign SAs to VMs
    - specify service instance during VM instance creation: `gcloud compute instances create [INSTANCE_NAME] --service-account [SERVICE_ACCOUNT_EMAIL]`
  - granting SA access to another project

## Stackdriver

1. install stackdriver agents
   - install stackdriver monitoring agent:
     - `curl -sSO https://dl.google.com/cloudagents/install-monitoring-agent.sh`
     - `sudo bash install-monitoring-agent.sh`
   - install stackdriver logging agent:
     - `curl -sSO https://dl.google.com/cloudagents/install-logging-agent.sh`
     - `sudo bash install-logging-agent.sh --structured`
2. stackdriver requires workspace to store data. Create workspace, select project.
3. setup metrics-based alerting
   - custom metrics name start with `custom.googleapis.com/`.
     - 2 methods of creation:
       1. OpenCensus (https://opencensus.io/) -- open source monitoring lib, higher level monitoring focused API
       2. stackdriver monitoring API -- lower level
     - custom metric args:
       - type name, unique within project
       - project
       - display name & description
       - metric kind (ex: gauge, delta, cumulative)
       - metric labels
       - monitored resource objectst o include w/ time series data points
   - [programmatically create custom metrics](https://cloud.google.com/monitoring/custom-metrics/creating-metrics)
 - Installing Stackdriver agents
   - cloud monitoring agent:
     - GCE VMs: optional but recommended. Monitoring can still access some metrics from VM's hypervisor without agent, such as CPU util, some disk traffic metrics, network traffic, uptime.
     - Amazon Elastic Compute Cloud (EC2) VMs: required
   - cloud logging agent: best practice to run on all VMs
     - GCE & Amazon EC2 VM images do NOT include logging agent, must be installed
     - GKE & GAE: agent included in VM image.

### Log Sinks

params:
- sink name
- sink service
  - BQ
  - GCS
  - pub/sub
    - data is base64 encoded
    - in LogEntry object structure
      - logname
      - timestamp
      - textPayload
      - resource properties: type, instance_id, zone, project_id
  - custom destination
- sink destination

filtering log messages:
- label or text search
- resource type
- log type
- log level
- time limit

Cloud trace helps devs & devops identify code sections that are performance bottlenecks
filtering log traces:
- report criteria
- time
- trace query
- HTTP method
- return status

cloud debug allows devs to insert log statements (without altering src code) or take snapshots of app state
- enabled by default on App Engine. Can be enabled for compute & k8s engines

## Calculators

- https://cloud.google.com/products/calculator/
-
## MISC.

- `gsutil acl ch -u coworker@email.domain:r gs://mybucket/myfile`
- `gcloud container clusters get-credentials`

## Drill Areas

- pricing calculator
  - GPUs: in GCE & GKE tabs. GAE doesn't support GPUs.
- Activity Page
  - filters:
    - user
    - activity types: billing, configuration, data access, development, monitoring
    - resource types
- GCE instance startup sequence:
  1. OS boot up -- instance is running
  2. gcloud completes (if ran without --async)
  3. metadata service -- provides startup script to OS boot process
  4. gsutil cmd gets metadata (ex: SA token) -- synchronous by default, will take time to transfer volumne of data to instance
  5. stackdriver agent pushes logs to show startup script progress thru-out (streamed)
  6. Once transfer completes, start script completes, more logs pushed to stackdriver
- kubectl cmds
- metadata-flavor: google header?
- key ID info:
  - SA email address =OR= project number to construct email, used by default GCE SA -- not ID like app engine
    - if want to use nonn-default SA: need both project ID (not number!) + SA name
- hashing & salting passwords? something about view, not just compare, and hashing making it unusable.
- **predefined machine types**
  - named by CPU counts -- always power of 2
  - custom machine types: can't add more RAM per CPU than `highmem` machine types (unless using extended memory -- which isn't available to 8-cpu custom type option bc no `--custom-extensions`)
- default SA permissions scopes: allows reading from GCS
- custom VMs can have up to 64 PUs and up to 6.4GB of RAM per vCPU
- Kubernetes reserves CPU capacity according to the following schedule:
  1. 6 percent of the first core
  2. 1 percent of the next core (up to two cores)
  3. 0.5 percent of the next two cores (up to four cores)
  4. 0.25 percent of any cores above four cores
- cmds that require zone:
  - gcloud container clusters describe --zone us-central1-a standard-cluster-1
  - gcloud container clusters get-credentials --zone us-central1-a standard-cluster-1
- Storage options chart
  - transactions
  - minimum storage duration
- logging
  - when to install stackdriver agent
  - stdout+stderr. (is writing to .log file ever necessary?)
- common iam policies and permissions

## Practice Exam Questions

- flash cards: https://quizlet.com/421219034/gcp-associate-cloud-engineer-flash-cards/
- https://www.vmexam.com/google/google-gcp-ace-certification-exam-sample-questions
- https://www.testpreptraining.com/google-cloud-certified-associate-cloud-engineer-free-practice-test
- https://gcp-examquestions.com/gcp-associate-cloud-engineer-practice-exam-part-1/
  - https://gcp-examquestions.com/gcp-associate-cloud-engineer-practice-exam-part-2/
  - https://gcp-examquestions.com/gcp-associate-cloud-engineer-practice-exam-part-3/
  - https://gcp-examquestions.com/gcp-associate-cloud-engineer-practice-exam-part-4/
  - https://gcp-examquestions.com/gcp-associate-cloud-engineer-practice-exam-part-5/
  - https://gcp-examquestions.com/gcp-associate-cloud-engineer-practice-exam-part-6/
- https://books.google.com/books?id=haxjEAAAQBAJ&pg=PT14&lpg=PT14&dq=%22man+gcloud+compute+instances+create%22&source=bl&ots=yOzwBdhYou&sig=ACfU3U2DwPr3kndWw-DO28wyi2EMB8WeaQ&hl=en&sa=X&ved=2ahUKEwjZrMzpmNv2AhWkjYkEHbAmDK0Q6AF6BAgCEAM#v=onepage&q=%22man%20gcloud%20compute%20instances%20create%22&f=false