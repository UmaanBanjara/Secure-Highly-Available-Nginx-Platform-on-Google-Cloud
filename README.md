## Project overview

This project demonstrates how to deploy a small highly available web platform without application code or containers. A public Nginx edge server accepts HTTPS traffic and distributes requests across two private Nginx backend servers.

The backend VMs have no public IP addresses. Administration uses Identity-Aware Proxy (IAP), outbound internet access uses Cloud NAT, and firewall rules restrict backend web traffic to the edge VM.

## Architecture

```text
                         Internet
                            |
                 HTTPS: nginx.umaanbanjara.com.np
                            |
                     GCP firewall rule
                       TCP 80 and 443
                            |
                  +--------------------+
                  | nginx-edge-vm      |
                  | 10.30.1.10         |
                  | Nginx + TLS        |
                  | Reverse proxy      |
                  | Load balancer      |
                  +--------------------+
                       /          \
              private HTTP     private HTTP
                    /                \
       +--------------------+  +--------------------+
       | nginx-backend-1    |  | nginx-backend-2    |
       | 10.30.2.10         |  | 10.30.2.11         |
       | No external IP     |  | No external IP     |
       +--------------------+  +--------------------+
                    \                /
                     Cloud NAT for outbound access
```

## Google Cloud resources

- Custom VPC with separate edge and backend subnets
- Public edge VM with a reserved static IP
- Two private backend VMs without external IP addresses
- Cloud Router and Cloud NAT for backend outbound connectivity
- IAP-based SSH access to the private backends
- Least-privilege VPC firewall rules using network tags
- Private Cloud Storage bucket for configuration backups
- Dedicated service account with scoped IAM permissions
- Cloud Monitoring uptime check and alert notification
- Google Cloud Ops Agent for host metrics and logging

## Nginx functionality

- HTTPS termination at the edge server
- Reverse proxying to private backend addresses
- Round-robin traffic distribution
- Automatic retry and failover when a backend becomes unavailable
- Preserved client and forwarding headers
- Custom access logs containing upstream address, status, and timing
- Separate error logging
- Per-client request rate limiting
- Security response headers
- Hidden Nginx version information

## Network security model

The platform follows a layered network design:

- Only the edge VM has a public IP address.
- Public traffic is allowed only on TCP ports 80 and 443.
- Direct SSH to the edge VM is restricted to an approved source address.
- Backend SSH is available through IAP only.
- Backend TCP port 80 accepts traffic only from instances tagged as edge servers.
- Backend VMs use Cloud NAT for package downloads without accepting inbound internet traffic.
- HTTP traffic is redirected permanently to HTTPS.

## TLS and DNS

The subdomain `nginx.umaanbanjara.com.np` points to the edge VM's static IP address.

Let's Encrypt and Certbot provide the TLS certificate. Certbot updates the Nginx configuration, redirects HTTP to HTTPS, and schedules automatic certificate renewal. Renewal was verified using a dry run.

## Logging and monitoring

The custom Nginx access log records:

- Client IP address
- HTTP request and status code
- Response size
