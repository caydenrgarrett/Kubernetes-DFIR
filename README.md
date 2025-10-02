# Kubernetes DFIR (Digital Forensics & Incident Response) <br>

![image alt](https://concisesoftware.com/wp-content/uploads/2020/01/Kubernetes-logo.png)

## Project Overview

This project documents a hands-on Digital Forensics and Incident Response exercise focused on investigating and responding to a security breach in a Kubernetes cluster. The exercise demonstrates practical DFIR techniques using logs, process analysis, and network forensics to identify, isolate, and remediate threats in containerized environments.

## Project Details

**Duration:** March 2025  
**Focus Area:** Kubernetes Security Incident Response  
**Source & Inspiration:** [TryHackMe](https://tryhackme.com) - Digital Forensics and Incident Response learning platform, and Cyberwox

## Scenario

A security breach was detected in a Kubernetes cluster environment. The incident required immediate investigation using digital forensics techniques to understand the attack vector, assess the damage, and implement appropriate remediation measures.

## Investigation Process

### 1. Initial Detection
- **Input:** Security alerts indicating suspicious activity in the Kubernetes cluster
- **Output:** Confirmed breach identification and initial threat assessment

### 2. Log Analysis
- **Input:** 
  - Kubernetes audit logs
  - Container runtime logs
  - Application logs
  - System logs from worker nodes
- **Output:** 
  - Timeline of malicious activities
  - Identified attack vectors and techniques used
  - Compromised resources and services

### 3. Process Analysis
- **Input:**
  - Running container processes
  - Pod configurations and states
  - Service account permissions
  - Network policies
- **Output:**
  - Malicious processes and containers identified
  - Privilege escalation paths discovered
  - Lateral movement techniques documented

### 4. Network Analysis
- **Input:**
  - Network traffic logs
  - DNS queries
  - Ingress/egress connections
  - Service mesh traffic (if applicable)
- **Output:**
  - Command and control (C2) communications identified
  - Data exfiltration attempts detected
  - Network-based attack patterns documented

### 5. Threat Isolation
- **Input:** Analysis findings from logs, processes, and network investigation
- **Output:**
  - Affected pods and containers quarantined
  - Malicious services disabled
  - Network access restrictions implemented
  - Compromised service accounts revoked

### 6. Forensic Evidence Collection
- **Input:** Compromised systems and artifacts
- **Output:**
  - Disk images of affected nodes
  - Container filesystem snapshots
  - Network packet captures
  - Configuration backups
  - Timeline documentation

### 7. Reporting and Recommendations
- **Input:** Complete investigation findings and evidence
- **Output:**
  - Detailed incident report
  - Security improvement recommendations
  - Updated security policies and procedures
  - Enhanced monitoring and detection capabilities

## Key Learning Outcomes

- **Kubernetes Security Fundamentals:** Understanding container security principles and common attack vectors
- **DFIR Methodologies:** Applying structured incident response processes to containerized environments
- **Log Analysis Techniques:** Parsing and correlating various log sources for threat detection
- **Network Forensics:** Analyzing container network traffic for malicious activities
- **Evidence Handling:** Proper collection and preservation of digital evidence in cloud-native environments

## Tools and Techniques Used

- **Log Analysis:** kubectl, fluentd, ELK stack
- **Process Monitoring:** ps, top, htop, kubectl get pods
- **Network Analysis:** tcpdump, Wireshark, kubectl port-forward
- **Forensic Collection:** dd, kubectl cp, docker save
- **Security Scanning:** kube-hunter, kube-bench, Falco

## Key Kubernetes DFIR Commands

### 1. Initial Assessment and Discovery

```bash
# Get cluster information
kubectl cluster-info
kubectl get nodes -o wide
kubectl version --short

# List all resources in the cluster
kubectl get all --all-namespaces
kubectl get pods --all-namespaces
kubectl get services --all-namespaces
kubectl get deployments --all-namespaces

# Check for suspicious or privileged pods
kubectl get pods --all-namespaces -o wide
kubectl describe pods --all-namespaces | grep -E "(SecurityContext|privileged|hostNetwork)"
```

### 2. Log Collection and Analysis

```bash
# Collect audit logs
kubectl get events --all-namespaces --sort-by='.lastTimestamp'
kubectl logs --previous <pod-name> -n <namespace>

# Export all pod logs
for pod in $(kubectl get pods --all-namespaces -o jsonpath='{range .items[*]}{.metadata.namespace}{" "}{.metadata.name}{"\n"}{end}'); do
  namespace=$(echo $pod | cut -d' ' -f1)
  name=$(echo $pod | cut -d' ' -f2)
  kubectl logs $name -n $namespace > logs/${namespace}_${name}.log
done

# Check system logs on nodes
kubectl get nodes
kubectl debug node/<node-name> -it --image=busybox -- chroot /host /bin/bash
```

### 3. Process and Container Analysis

```bash
# Get detailed pod information
kubectl get pods -o yaml --all-namespaces
kubectl describe pod <pod-name> -n <namespace>

# Check running processes in containers
kubectl exec -it <pod-name> -n <namespace> -- ps aux
kubectl exec -it <pod-name> -n <namespace> -- top
kubectl exec -it <pod-name> -n <namespace> -- netstat -tulpn

# Check container runtime information
kubectl get pods -o jsonpath='{.items[*].spec.containers[*].image}' --all-namespaces
docker ps -a  # On worker nodes
crictl ps -a  # For CRI-O runtime
```

### 4. Network Analysis

```bash
# Check network policies
kubectl get networkpolicies --all-namespaces
kubectl describe networkpolicy <policy-name> -n <namespace>

# Analyze network connections
kubectl exec -it <pod-name> -n <namespace> -- netstat -an
kubectl exec -it <pod-name> -n <namespace> -- ss -tulpn
kubectl exec -it <pod-name> -n <namespace> -- lsof -i

# Port forwarding for deeper analysis
kubectl port-forward <pod-name> -n <namespace> 8080:8080
```

### 5. Security Context and RBAC Analysis

```bash
# Check security contexts
kubectl get pods --all-namespaces -o jsonpath='{range .items[*]}{.metadata.namespace}{" "}{.metadata.name}{" "}{.spec.securityContext}{"\n"}{end}'

# Analyze RBAC permissions
kubectl get roles --all-namespaces
kubectl get clusterroles
kubectl get rolebindings --all-namespaces
kubectl get clusterrolebindings
kubectl describe clusterrolebinding <binding-name>

# Check service accounts
kubectl get serviceaccounts --all-namespaces
kubectl describe serviceaccount <sa-name> -n <namespace>
```

### 6. Resource and Configuration Analysis

```bash
# Check resource usage
kubectl top nodes
kubectl top pods --all-namespaces
kubectl describe nodes

# Analyze configurations
kubectl get configmaps --all-namespaces
kubectl get secrets --all-namespaces
kubectl get persistentvolumes
kubectl get persistentvolumeclaims --all-namespaces

# Check for suspicious configurations
kubectl get pods --all-namespaces -o jsonpath='{range .items[*]}{.metadata.namespace}{" "}{.metadata.name}{" "}{.spec.hostNetwork}{" "}{.spec.privileged}{"\n"}{end}'
```

### 7. Threat Isolation and Containment

```bash
# Isolate suspicious pods
kubectl delete pod <pod-name> -n <namespace>
kubectl scale deployment <deployment-name> --replicas=0 -n <namespace>

# Apply network policies for isolation
kubectl apply -f - <<EOF
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
  namespace: <namespace>
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
EOF

# Revoke service account tokens
kubectl delete secret <secret-name> -n <namespace>
```

### 8. Forensic Evidence Collection

```bash
# Copy files from containers
kubectl cp <namespace>/<pod-name>:/path/to/file ./forensic_evidence/

# Export pod specifications
kubectl get pod <pod-name> -n <namespace> -o yaml > forensic_evidence/pod_<pod-name>.yaml

# Create snapshots of container filesystems
kubectl exec <pod-name> -n <namespace> -- tar czf /tmp/forensic_archive.tar.gz /
kubectl cp <namespace>/<pod-name>:/tmp/forensic_archive.tar.gz ./forensic_evidence/

# Export all relevant resources
kubectl get all,secrets,configmaps,networkpolicies -n <namespace> -o yaml > forensic_evidence/namespace_<namespace>_complete.yaml
```

### 9. Security Scanning and Assessment

```bash
# Run kube-hunter for security scanning
kubectl run kube-hunter --image=aquasec/kube-hunter:latest --rm -it --restart=Never -- pod

# Run kube-bench for CIS benchmark testing
kubectl run kube-bench --image=aquasec/kube-bench:latest --rm -it --restart=Never -- job

# Check for security policies
kubectl get podsecuritypolicies
kubectl get securitycontextconstraints  # For OpenShift
```

### 10. Monitoring and Alerting

```bash
# Set up monitoring for ongoing security
kubectl get pods -n monitoring
kubectl logs -n monitoring deployment/prometheus

# Check Falco security monitoring
kubectl logs -n falco deployment/falco
kubectl get pods -n falco

# Review audit logs
kubectl get events --all-namespaces --field-selector type=Warning
```

### 11. Cleanup and Remediation

```bash
# Clean up malicious resources
kubectl delete deployment <malicious-deployment> -n <namespace>
kubectl delete service <malicious-service> -n <namespace>
kubectl delete configmap <malicious-configmap> -n <namespace>
kubectl delete secret <malicious-secret> -n <namespace>

# Restore from backups
kubectl apply -f backup/namespace_<namespace>_clean.yaml

# Verify cluster health
kubectl get nodes
kubectl get pods --all-namespaces
kubectl cluster-info
```

## Security Improvements Implemented

Based on the investigation findings, the following security measures were enhanced:

1. **Enhanced Monitoring:** Implemented comprehensive logging and alerting
2. **Network Policies:** Strengthened network segmentation and access controls
3. **RBAC:** Tightened role-based access controls and least privilege principles
4. **Image Security:** Enhanced container image scanning and vulnerability management
5. **Runtime Protection:** Deployed runtime security solutions for threat detection

## Acknowledgments

This project was inspired by and based on learning materials from **TryHackMe**, a leading platform for cybersecurity education and hands-on learning experiences. TryHackMe provides excellent resources for developing practical skills in digital forensics, incident response, and cloud security.

- **TryHackMe Platform:** https://tryhackme.com
- **DFIR Learning Path:** Digital Forensics and Incident Response modules
- **Kubernetes Security:** Container and orchestration security challenges

## Project Structure

```
kubernetes-dfir/
├── README.md                 # This file
├── investigation/            # Investigation artifacts and findings
│   ├── logs/                # Collected log files
│   ├── evidence/            # Forensic evidence
│   └── reports/             # Investigation reports
├── tools/                   # Custom scripts and tools used
└── documentation/           # Additional documentation and references
```

## Future Enhancements

- Automation of common DFIR tasks in Kubernetes environments
- Integration with SIEM platforms for real-time threat detection
- Development of custom detection rules and signatures
- Creation of incident response playbooks for Kubernetes environments

---

*This project demonstrates practical application of DFIR principles in modern containerized environments, with special thanks to TryHackMe for providing the foundational knowledge and inspiration.*
