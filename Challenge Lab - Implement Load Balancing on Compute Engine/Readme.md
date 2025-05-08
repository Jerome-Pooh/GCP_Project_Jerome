# Implement Load Balancing on Compute Engine: Challenge Lab

## Overview

This project demonstrates a solution to the **Implement Load Balancing on Compute Engine: Challenge Lab**. The objective is to provision resources that support an HTTP Load Balancer in front of a managed instance group running Nginx web servers, and to configure a jumphost instance for administrative access.

## Tasks Summary

1. Set default region and zone.
2. Create a jumphost instance.
3. Set up a startup script to install and configure Nginx.
4. Create an instance template using the startup script.
5. Create a managed instance group.
6. Configure firewall rules.
7. Create a health check.
8. Create a backend service and attach the instance group.
9. Configure URL map and proxy.
10. Set up forwarding rule.

---

## Set the Default Region and Zone

Set the default region and zone to simplify commands:

```bash
gcloud config set compute/region Region
gcloud config set compute/zone Zone
````

---

## Task 1: Create a Project Jumphost Instance

You will use this instance to perform maintenance for the project.
Requirements:
• Name the instance Instance name.
• Create the instance in the ZONE zone.
• Use an e2-micro machine type.
Use the default image type (Debian Linux).

---

## Task 2: Set Up an HTTP Load Balancer

### Create Startup Script

This script will install and start Nginx, and customize the default index page.

```bash
cat << EOF > startup.sh
#! /bin/bash
apt-get update
apt-get install -y nginx
service nginx start
sed -i -- 's/nginx/Google Cloud Platform - '"\$HOSTNAME"'/' /var/www/html/index.nginx-debian.html
EOF
```

---

### Create an Instance Template

```bash
gcloud compute instance-templates create Global-template \
  --region=Region \
  --network=default \
  --subnet=default \
  --tags=allow-health-check \
  --machine-type=e2-medium \
  --image-family=debian-11 \
  --image-project=debian-cloud \
  --metadata-from-file startup-script=startup.sh
```

---

### Create a Managed Instance Group

```bash
gcloud compute instance-groups managed create lb-backend-group \
  --template=Global-template \
  --size=1 \
  --zone=Zone
```

---

### Create Firewall Rule

Allow HTTP traffic to reach the instances.

```bash
gcloud compute firewall-rules create firewall-rule \
  --target-tags network-lb-tag \
  --allow tcp:80
```

---

### Create a Health Check

```bash
gcloud compute health-checks create http http-basic-check \
  --port 80
```

---

### Create a Backend Service

```bash
gcloud compute backend-services create web-backend-service \
  --protocol=HTTP \
  --port-name=http \
  --health-checks=http-basic-check \
  --global
```

---

### Add Instance Group to Backend

```bash
gcloud compute backend-services add-backend web-backend-service \
  --instance-group=lb-backend-group \
  --instance-group-zone=Zone \
  --global
```

---

### Create a URL Map

```bash
gcloud compute url-maps create web-map-http \
  --default-service web-backend-service
```

---

### Create Target HTTP Proxy

```bash
gcloud compute target-http-proxies create http-lb-proxy \
  --url-map web-map-http
```

---

### Reserve Global Static IP Address

```bash
gcloud compute addresses create lb-ipv4-1 \
  --ip-version=IPV4 \
  --global
```

---

### Create Forwarding Rule

```bash
gcloud compute forwarding-rules create http-content-rule \
  --address=lb-ipv4-1 \
  --global \
  --target-http-proxy=http-lb-proxy \
  --ports=80
```

---

## Test the Load Balancer

To test load balancing and verify backend rotation:

```bash
gcloud compute addresses describe lb-ipv4-1 \
  --format="get(address)" \
  --global
```

Then:

```bash
while true; do curl -m1 http://<LB-IP>; done
```

---

## Conclusion

You have successfully implemented an HTTP Load Balancer in front of a managed instance group, using GCP best practices for resource provisioning and configuration. This lab demonstrates your ability to build scalable and reliable cloud infrastructure using Google Cloud Platform.

```

---
