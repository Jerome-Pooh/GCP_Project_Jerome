# Set Up Network and Application Load Balancers

This project demonstrates how to set up and configure both Network Load Balancers and Application Load Balancers on Google Cloud Platform using Compute Engine virtual machines (VMs).

## Overview

In this project, you will learn the differences between Network Load Balancers (Layer 4) and Application Load Balancers (Layer 7), and how to deploy them on GCP to distribute traffic across VM instances. You'll create multiple web servers, configure firewall rules, and verify traffic distribution using static IPs and health checks.

## Objectives

* Set the default region and zone for resources.
* Create multiple web server instances.
* Configure a Network Load Balancer.
* Set up traffic routing using a forwarding rule.
* Deploy an Application Load Balancer with managed instance groups.
* Test load balancer functionality.

# Set Up Network and Application Load Balancers

This lab demonstrates how to set up both **Network Load Balancers (L4)** and **Application Load Balancers (L7)** on Google Cloud using Compute Engine VM instances, managed instance groups, and other core networking services.

---

## Task 1: Set the default region and zone for all resources

Set the default region:

```bash
gcloud config set compute/region REGION
```

Set the default zone:

```bash
gcloud config set compute/zone ZONE
```

> 💡 Learn more: [Compute Engine's Regions and Zones](https://cloud.google.com/compute/docs/regions-zones)

---

## Task 2: Create multiple web server instances

You’ll create 3 VMs, each with Apache installed and a unique home page.

### Create VM: www1

```bash
gcloud compute instances create www1 \
  --zone=ZONE \
  --tags=network-lb-tag \
  --machine-type=e2-small \
  --image-family=debian-11 \
  --image-project=debian-cloud \
  --metadata=startup-script='#!/bin/bash
    apt-get update
    apt-get install apache2 -y
    service apache2 restart
    echo "<h3>Web Server: www1</h3>" | tee /var/www/html/index.html'
```

### Create VM: www2

```bash
gcloud compute instances create www2 \
  --zone=ZONE \
  --tags=network-lb-tag \
  --machine-type=e2-small \
  --image-family=debian-11 \
  --image-project=debian-cloud \
  --metadata=startup-script='#!/bin/bash
    apt-get update
    apt-get install apache2 -y
    service apache2 restart
    echo "<h3>Web Server: www2</h3>" | tee /var/www/html/index.html'
```

### Create VM: www3

```bash
gcloud compute instances create www3 \
  --zone=ZONE \
  --tags=network-lb-tag \
  --machine-type=e2-small \
  --image-family=debian-11 \
  --image-project=debian-cloud \
  --metadata=startup-script='#!/bin/bash
    apt-get update
    apt-get install apache2 -y
    service apache2 restart
    echo "<h3>Web Server: www3</h3>" | tee /var/www/html/index.html'
```

### Create a firewall rule

```bash
gcloud compute firewall-rules create www-firewall-network-lb \
  --target-tags network-lb-tag --allow tcp:80
```

### Verify VMs

```bash
gcloud compute instances list
curl http://[IP_ADDRESS]
```

---

## Task 3: Configure the load balancing service

### Create static external IP

```bash
gcloud compute addresses create network-lb-ip-1 \
  --region REGION
```

### Create HTTP health check

```bash
gcloud compute http-health-checks create basic-check
```

### Create a target pool with health check

```bash
gcloud compute target-pools create www-pool \
  --region REGION --http-health-check basic-check
```

### Add VMs to target pool

```bash
gcloud compute target-pools add-instances www-pool \
  --instances www1,www2,www3
```

### Create forwarding rule

```bash
gcloud compute forwarding-rules create www-rule \
  --region REGION \
  --ports 80 \
  --address network-lb-ip-1 \
  --target-pool www-pool
```

---

## Task 4: Send traffic to your instances

```bash
gcloud compute forwarding-rules describe www-rule --region REGION
```

```bash
IPADDRESS=$(gcloud compute forwarding-rules describe www-rule --region REGION --format="json" | jq -r .IPAddress)
echo $IPADDRESS
```

```bash
while true; do curl -m1 $IPADDRESS; done
```

> Use `Ctrl+C` to stop the loop.

---

## Task 5: Create an Application Load Balancer

### Create instance template

```bash
gcloud compute instance-templates create lb-backend-template \
  --region=REGION \
  --network=default \
  --subnet=default \
  --tags=allow-health-check \
  --machine-type=e2-medium \
  --image-family=debian-11 \
  --image-project=debian-cloud \
  --metadata=startup-script='#!/bin/bash
    apt-get update
    apt-get install apache2 -y
    a2ensite default-ssl
    a2enmod ssl
    vm_hostname="$(curl -H "Metadata-Flavor:Google" http://169.254.169.254/computeMetadata/v1/instance/name)"
    echo "Page served from: $vm_hostname" | tee /var/www/html/index.html
    systemctl restart apache2'
```

### Create a managed instance group

```bash
gcloud compute instance-groups managed create lb-backend-group \
  --template=lb-backend-template --size=2 --zone=ZONE
```

### Create firewall rule for health checks

```bash
gcloud compute firewall-rules create fw-allow-health-check \
  --network=default \
  --action=allow \
  --direction=ingress \
  --source-ranges=130.211.0.0/22,35.191.0.0/16 \
  --target-tags=allow-health-check \
  --rules=tcp:80
```

### Create global static IP

```bash
gcloud compute addresses create lb-ipv4-1 \
  --ip-version=IPV4 --global

gcloud compute addresses describe lb-ipv4-1 \
  --format="get(address)" --global
```

### Create health check

```bash
gcloud compute health-checks create http http-basic-check \
  --port 80
```

### Create backend service

```bash
gcloud compute backend-services create web-backend-service \
  --protocol=HTTP \
  --port-name=http \
  --health-checks=http-basic-check \
  --global
```

### Add backend instance group

```bash
gcloud compute backend-services add-backend web-backend-service \
  --instance-group=lb-backend-group \
  --instance-group-zone=ZONE \
  --global
```

### Create URL map

```bash
gcloud compute url-maps create web-map-http \
  --default-service web-backend-service
```

### Create target HTTP proxy

```bash
gcloud compute target-http-proxies create http-lb-proxy \
  --url-map web-map-http
```

### Create global forwarding rule

```bash
gcloud compute forwarding-rules create http-content-rule \
  --address=lb-ipv4-1 \
  --global \
  --target-http-proxy=http-lb-proxy \
  --ports=80
```

---

## Task 6: Test traffic sent to your instances

1. In Google Cloud Console, search for **Load balancing**.
2. Open **web-map-http**.
3. Verify backend VMs are marked **Healthy**.
4. Navigate to `http://<IP_ADDRESS>` in your browser.
5. Refresh multiple times to see content from different backends.

---

## 🎉 Congratulations!

You've successfully built:

* A **Network Load Balancer** using unmanaged VMs and target pools.
* An **Application Load Balancer** using a managed instance group and HTTP proxy.
