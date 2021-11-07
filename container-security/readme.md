# Container Security Expert

# Container Reconnaissance

- Overview of Container Security
- Attack surface of the container ecosystem
- Analysis of the attack surface
- Using native tools
- Using third-party tools
- Identifying the components and their security state
- Get an inventory of containers
- Environment variables
- Docker volumes
- Networking
- Ports used/Port forwarding
- Capabilities and namespaces in Docker

# Hands-on Exercises:

- Scanning the remote host for unauthenticated Docker API access
- Identify a container and extract sensitive information
- Identify misconfigurations in namespace, capabilities, and networking
- Create and restore a snapshot(tar) of the container for further analysis


#  Attacking Containers and Containerized Apps

- Image-based attacks
- Malicious Images
- Extracting passwords, tokens, TLS certs, etc.,
- Exploiting vulnerable components
- Registry-based attacks
- Insecure Docker registries
- Open Docker registries
- Lack of authorization (RBAC)
- Container-based attacks
- Manipulating the Privileged mode containers
- Attacking mounted docker volumes
- Abusing SetUID/SetGID binaries
- Exploiting shared namespaces
- Attacking Linux capabilities
- Docker host (Daemon) / kernel attacks
- Exploiting unauthenticated Docker API
- Insecure Docker endpoint
- Lack of network segregation
- Denial of service attacks
- Kernel exploits
- Privilege escalation methods in Docker
- Security misconfigurations
- Attacking management tools (Portainer)
- Exploiting OWASP Top 10 issues in containerized apps

# Defending Containers and Containerized Apps on Scale
- Container image security
- Building secure container images
- Choosing base images
- Distroless images
- Scratch images
- Security Linting of Dockerfiles
- Static Analysis of container images
- Static Analysis library for container
- Docker host security configurations
- Kernel Hardening using SecComp and AppArmor
- Custom policy creation using SecComp and AppArmor
- Docker Daemon security configurations
- Docker user remapping
- Docker runtime security (gVisor, Kata)
- Docker socket configuration 
 - fd
 - TCP socket
 - TLS authentication
- Dynamic Analysis of the container hosts and daemons
- Network Security in containers
- Segregating networks
- Misc Docker Security Configurations
- Content Trust and Integrity checks
- Docker Registry security configurations
- Internal vs. Public Registries
- Authentication and Authorization (RBAC)
- Image scanning
- Policy enforcement
- DevOps CI/CD Integration
- Docker Tools, Techniques and Tactics
- Tools
- Dive
- Dockle
- Techniques
- Tactics
# Hands-on Exercises:
- Minimize security misconfigurations in Docker with CIS
- Build a secure & most miniature image to minimize the footprint
- Build a distro less image to reduce the footprint
- Docker Content Trust with Notary
- Securing the container by default using Harbor
- Scanning Docker for vulnerabilities with terrascan

# Security Monitoring of Containers
- Monitoring and incident response in containers
- Docker events
- Docker logs
- Docker runtime prevention
- Security monitoring using Wazuh
- Policy creation, enforcement, and management
# Hands-on Exercises:

- VMWare Harbor – Securing Docker image with Harbor
- Tracee – Runtime security
