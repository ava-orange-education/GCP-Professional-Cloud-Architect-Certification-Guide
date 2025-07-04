# GCP Hands-On Lab: Managed Instance Groups and HTTP Load Balancing

This lab walks you through setting up a scalable, highly available application using Managed Instance Groups (MIGs) and an HTTP Load Balancer in Google Cloud.  
We'll use `gcloud` commands, and assume you have already setup your free account according to the lab steps of Chapter 1. Please replace your Project ID below.
Also please run the following commands in the Google Cloud Shell for an easy and a seamless experience

---

## 🛠️ Step 1: Set Default Project and Region

```bash
gcloud config set project <YOUR-PROJECT-ID>
gcloud config set compute/region us-central1
gcloud config set compute/zone us-central1-a
```

> These commands set the project, default region, and zone for all subsequent `gcloud` commands, so you don’t have to specify them repeatedly.

---

## 🛠️ Step 2: Create a Startup Script

```bash
cat <<EOF > startup-script.sh
#! /bin/bash
apt-get update
apt-get install -y apache2
systemctl start apache2
echo "<h1>Welcome to MIG Instance</h1>" > /var/www/html/index.html
EOF
```

> This startup script installs Apache2, starts the web server, and writes a simple HTML page. It will be executed when the VM starts.

---

## 🛠️ Step 3: Create an Instance Template

```bash
gcloud compute instance-templates create web-template   --metadata-from-file startup-script=startup-script.sh   --machine-type e2-micro   --tags http-server   --image-family debian-11   --image-project debian-cloud
```

> Instance templates define the configuration for VM instances, such as machine type, disk image, metadata, and network tags. These templates are used by MIGs to create consistent VMs.

---

## 🛠️ Step 4: Create a Managed Instance Group (MIG)

```bash
gcloud compute instance-groups managed create web-mig   --base-instance-name web-instance   --size 2   --template web-template   --zone us-central1-a
```

> Creates a MIG named `web-mig` with 2 VM instances based on the `web-template`. The MIG will manage these VMs, ensuring availability and enabling autohealing/autoscaling.

---

## 🛠️ Step 5: Create a Health Check

```bash
gcloud compute health-checks create http basic-check   --port 80
```

> Defines a health check that periodically sends HTTP requests to port 80 of instances. This helps determine if instances are healthy and should receive traffic.

---

## 🛠️ Step 6: Attach Autohealing Policy to MIG

```bash
gcloud compute instance-groups managed update web-mig   --health-check basic-check   --initial-delay 300
```

> Attaches the health check to the MIG and specifies a 300-second initial delay before health checking begins. This allows new instances time to initialize.

---

## 🛠️ Step 7: Configure Autoscaling for MIG

```bash
gcloud compute instance-groups managed set-autoscaling web-mig   --max-num-replicas 5   --min-num-replicas 2   --target-cpu-utilization 0.6   --cool-down-period 90
```

> Enables autoscaling based on CPU usage. The MIG will keep between 2 and 5 instances, aiming for 60% average CPU usage, with a 90-second cool-down between scaling actions.

---

## 🛠️ Step 8: Create Firewall Rule to Allow HTTP

```bash
gcloud compute firewall-rules create allow-http   --target-tags http-server   --allow tcp:80
```

> Opens port 80 (HTTP) to allow external traffic to reach instances tagged with `http-server`.

---

## 🛠️ Step 9: Reserve a Global Static IP Address

```bash
gcloud compute addresses create web-ip   --ip-version=IPV4   --global
```

> Allocates a static global IP address for the HTTP(S) Load Balancer front-end, ensuring a consistent IP across regions.

---

## 🛠️ Step 10: Create a Backend Service and Add MIG

```bash
gcloud compute backend-services create web-backend-service   --protocol=HTTP   --health-checks=basic-check   --global

gcloud compute backend-services add-backend web-backend-service   --instance-group=web-mig   --instance-group-zone=us-central1-a   --global
```

> The backend service acts as an abstraction layer for your MIG, and connects it to the load balancer. It uses the health check to send traffic only to healthy VMs.

---

## 🛠️ Step 11: Create URL Map and Target Proxy

```bash
gcloud compute url-maps create web-map   --default-service web-backend-service

gcloud compute target-http-proxies create web-proxy   --url-map web-map
```

> A URL map routes incoming traffic to appropriate backend services based on path rules. The target proxy receives client HTTP requests and uses the URL map to forward them.

---

## 🛠️ Step 12: Create Forwarding Rule

```bash
gcloud compute forwarding-rules create http-content-rule   --address=web-ip   --global   --target-http-proxy=web-proxy   --ports=80
```

> The forwarding rule ties the static IP to the proxy, listens on port 80, and directs traffic to the load balancer.

---

## ✅ Step 13: Test the Setup

```bash
gcloud compute addresses describe web-ip --global
```

> Retrieves the external IP address. Open it in your browser to view the Apache web server running on the MIG-backed instances.

---

## 🧹 Step 14: Cleanup Resources

```bash
gcloud compute forwarding-rules delete http-content-rule --global --quiet
gcloud compute target-http-proxies delete web-proxy --quiet
gcloud compute url-maps delete web-map --quiet
gcloud compute backend-services delete web-backend-service --global --quiet
gcloud compute instance-groups managed delete web-mig --zone us-central1-a --quiet
gcloud compute health-checks delete basic-check --quiet
gcloud compute instance-templates delete web-template --quiet
gcloud compute firewall-rules delete allow-http --quiet
gcloud compute addresses delete web-ip --global --quiet
rm startup-script.sh
```

> This cleans up all the resources created during the lab to avoid unnecessary billing.

---

**✅ Lab Complete!**  
You now have a working knowledge of deploying scalable and highly available applications using GCP Managed Instance Groups and Load Balancing.