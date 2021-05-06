This lab walks you through GCP Virtual Private Cloud (VPC) creation using Custom subnet creation mode.

# Lab Tasks:
1. Login into GCP Console.
2. Creating a VPC using the Custom mode.
3. Choosing Private Google Access.
4. Creating a VM Instance and Test.

## Login into GCP Console
Run the following from the Cloud Terminal:

- List and check if you are authenticated
  ```
  gcloud auth list
  ```
- To authenticate with a user identity (via web flow) which then authorizes gcloud and other SDK tools to access Google Cloud Platform.
  ```
  gcloud auth login
  ```
- To get the project id
  ```
  export PROJECTID=$(gcloud info --format='value(config.project)')
  ```

## Creating a VPC using the Custom mode
Run the following from the Cloud Terminal:
```
gcloud compute networks create codelabs-vpc \ 
--subnet-mode=custom \
--description="codelabs virtual private cloud" \
--bgp-routing-mode=regional \ 
--mtu=1460
```
List the networks in project
```
gcloud compute networks list
```
Describe the newly created network
```
gcloud compute networks describe codelabs-vpc
```

## Choosing Private Google Access
Create a new subnet manually.

Run the following from the Cloud Terminal:
```
gcloud compute networks subnets create codelabs-subnet1a \
--network=codelabs-vpc \
--range=10.0.2.0/29 \
--region=us-central1 \
--enable-private-ip-google-access \
--no-enable-flow-logs
```
Verify if private google access is enabled
```
gcloud compute networks subnets describe codelabs-subnet1a \
    --region=REGION \
    --format="get(privateIpGoogleAccess)"
```
Verify if VPC flow logs are disabled
```
gcloud compute networks subnets list \
  --project $PROJECTID \
  --filter="network=NETWORK_URL" \
  --format="csv(name,logConfig.enable)"
```
## Creating a VM Instance and Test

Add firewall rule to allow egress trafic
```
gcloud compute firewall-rules create codelabs-allow-egress-all \ 
--description "allow egress traffic to all" \
--no-enable-logging \
--network codelabs-vpc \
--priority 1000 \
--direction egress \
--action ALLOW \
--rules all \
--destination-ranges 0.0.0.0/0 
```

Add firewall rule to allow ssh 
```
gcloud compute firewall-rules create codelabs-allow-ingress-ssh \
--description "allow ingress ssh to all vm's in codelabs-vpc" \
--no-enable-logging \
--network "codelabs-vpc" \
--direction "ingress" \
--rules "tcp:22" \
--action ALLOW \
--priority 1000 \
--source-ranges 0.0.0.0/0
```

Check if Firewall rule is add 

```
gcloud compute firewall-rules list \
--filter network=codelabs-vpc \
--sort-by priority \ 
--format="table( name, 
    network, 
    direction, 
    priority, 
    sourceRanges.list():label=SRC_RANGES, 
    destinationRanges.list():label=DEST_RANGES, 
    allowed[].map().firewall_rule().list():label=ALLOW, 
    denied[].map().firewall_rule().list():label=DENY, 
    sourceTags.list():label=SRC_TAGS, 
    targetTags.list():label=TARGET_TAGS 
    )" 
```

Creating a VM in codelabs-vpc and codelabs-subnet1a

```
gcloud compute instances create codelabs-vm1 \
--network codelabs-vpc \
--subnet codelabs-subnet1a \
--region us-central1 \
--zone us-central1-a \
--image debian-10-buster-v20200309 \
--image-project student-00006 \
--machine-type n1-standard-1
```

SSH into the vm 

```
gcloud compute ssh \
--project student-00006 \
--zone us-central1-a \
codelabs-vm1
```

Once inside the vm enter `gsutils ls`

## Cleanup

Delete the instance 
```
gcloud compute instances delete codelabs-vm1 
```
Delete the firewall rule 
```
gcloud compute firewall-rules delete codelabs-allow-ingress-ssh
gcloud compute firewall-rules delete codelabs-allow-egress-all 
```
Delete the subnet
```
gcloud compute networks subnets delete codelabs-subnet1a
```
Delete the vpc network 
```
gcloud compute networks delete codelabs-vpc
```