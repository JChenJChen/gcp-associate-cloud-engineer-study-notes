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

## GCP Billing
- Owning a project & being able to control its resources is completely separate from being able to control the billing account tied to that project → admin account that created 2 projects can see 3 projects but not access the 1 project that’s created by user.

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

## VM

- gcloud config get-value project
- gcloud compute instances list
- gcloud services list
    - gcloud services list --enabled
    - gcloud services list --available
    - gcloud services list --available | grep compute
    - gcloud services enable compute.googleapis.com
        - enabling an API may enable other dependent or related APIs as well
    - gcloud compute instances create myvm