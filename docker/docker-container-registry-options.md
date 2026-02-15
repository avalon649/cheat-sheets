# Self-Hosted Docker Container Registry Options

A comprehensive guide to self-hosted Docker container registry solutions for managing and distributing container images on your own infrastructure.

---

## Open Source / Free

### Docker Registry / Distribution

- **Website**: <https://github.com/distribution/distribution>
- **License**: Apache 2.0
- **CNCF Project**: Yes (Graduated)

The official open-source container registry implementation donated to the CNCF. It implements the OCI Distribution Specification and serves as the core technology behind Docker Hub, GitHub Container Registry, GitLab Container Registry, and DigitalOcean Container Registry.

**Key Features:**
- Lightweight and minimal footprint
- OCI-compliant image storage and distribution
- Configurable storage backends (filesystem, S3, Azure Blob, GCS)
- Webhook notifications
- TLS and basic authentication support

**Best For:** Teams wanting a simple, no-frills registry with minimal overhead.

---

### Harbor

- **Website**: <https://goharbor.io/>
- **License**: Apache 2.0
- **CNCF Project**: Yes (Graduated)

A full-featured enterprise-grade container registry with built-in security and management capabilities. Designed as a cloud-native registry with first-class Kubernetes support.

**Key Features:**
- Role-based access control (RBAC)
- Vulnerability scanning (Trivy integration)
- Image signing and trust (Cosign/Notary)
- Multi-registry replication
- Helm chart repository
- Garbage collection
- Audit logging
- LDAP/AD/OIDC authentication
- Tag retention policies
- Robot accounts for CI/CD

**Deployment Options:**
- Docker Compose
- Helm Chart (Kubernetes)

**Best For:** Teams needing a comprehensive, production-ready registry with security scanning and access control.

---

### Portus

- **Website**: <http://port.us.org/>
- **License**: Apache 2.0

An authorization service and web UI front-end for the Docker Registry. Provides fine-grained access control over registries, namespaces, and repositories.

**Key Features:**
- Web-based UI for registry management
- Fine-grained access control (per namespace/repository)
- LDAP authentication
- Audit logging
- Search functionality
- Webhook support

**Note:** Portus development has slowed; consider Harbor as a more actively maintained alternative.

**Best For:** Teams wanting a UI and access control layer on top of Docker Registry.

---

### Gitea

- **Website**: <https://gitea.io/>
- **License**: MIT

A lightweight, self-hosted Git service that includes a built-in OCI-compliant container registry alongside code hosting and CI/CD capabilities.

**Key Features:**
- Integrated container registry with Git hosting
- OCI-compliant
- Supports Docker images and Helm charts
- Lightweight resource usage
- Built-in CI/CD (Gitea Actions)
- Package registry (npm, PyPI, Maven, etc.)

**Best For:** Teams wanting an all-in-one Git hosting and container registry solution with minimal resources.

---

### GitLab Container Registry

- **Website**: <https://docs.gitlab.com/ee/user/packages/container_registry/>
- **License**: MIT (Community Edition)

Built into self-hosted GitLab instances, providing tight integration with GitLab CI/CD pipelines and project management.

**Key Features:**
- Integrated with GitLab CI/CD pipelines
- Per-project and per-group registries
- Cleanup policies for tag expiration
- Vulnerability scanning (Ultimate tier)
- Built-in authentication via GitLab

**Best For:** Teams already using self-hosted GitLab for source code management and CI/CD.

---

### Distr

- **Website**: <https://distr.sh/>
- **License**: Open Source

An OCI-compliant artifact registry designed for software vendors to distribute container images and other artifacts to customers.

**Key Features:**
- OCI-compliant artifact registry
- Customer-facing distribution workflows
- Self-hostable
- License management integration

**Best For:** Software vendors needing to distribute container images to external customers.

---

### Zot

- **Website**: <https://zotregistry.dev/>
- **License**: Apache 2.0

A vendor-neutral, OCI-native container registry built from the ground up for OCI artifacts. Focused on simplicity and standards compliance.

**Key Features:**
- OCI-native (images and artifacts)
- Single binary deployment
- Built-in UI (zui)
- Image signature verification (Cosign/Notation)
- Vulnerability scanning
- Sync/mirroring from upstream registries
- Deduplication and garbage collection
- S3-compatible storage backend

**Best For:** Teams wanting a modern, standards-focused registry with minimal dependencies.

---

### Dragonfly

- **Website**: <https://d7y.io/>
- **License**: Apache 2.0
- **CNCF Project**: Yes (Incubating)

A P2P-based image and file distribution system designed for large-scale container image delivery. Acts as a distribution layer rather than a standalone registry.

**Key Features:**
- Peer-to-peer image distribution
- Reduces registry bandwidth usage
- Preheating/prefetching of images
- Compatible with any OCI registry
- Designed for large-scale clusters

**Best For:** Large-scale environments needing efficient image distribution across many nodes.

---

### Project Quay

- **Website**: <https://www.projectquay.io/>
- **License**: Apache 2.0

The open-source upstream project behind Red Hat Quay. Provides a full-featured container registry with security scanning and geo-replication.

**Key Features:**
- Vulnerability scanning (Clair integration)
- Geo-replication
- Repository mirroring
- Fine-grained access control
- Audit logging
- Automated image builds
- Helm chart repository
- LDAP/OAuth/OIDC authentication
- Tag expiration policies

**Best For:** Teams wanting a feature-rich open-source registry, especially in Red Hat/OpenShift environments.

---

## Commercial / Freemium (Self-Hosted Available)

### JFrog Container Registry

- **Website**: <https://jfrog.com/container-registry/>
- **Pricing**: Free standalone self-hosted edition; paid enterprise tiers

A universal artifact manager with Docker and Helm support. The free self-hosted edition provides core container registry functionality.

**Key Features:**
- Docker and Helm chart support
- Remote repository proxying/caching
- Virtual repositories (aggregate multiple registries)
- Replication
- REST API
- Fine-grained permissions
- Part of the broader JFrog Platform (Artifactory, Xray)

**Best For:** Teams needing multi-format artifact management beyond just containers.

---

### Mirantis Secure Registry

- **Website**: <https://www.mirantis.com/>
- **Pricing**: Commercial

Formerly Docker Trusted Registry (DTR). An enterprise-grade container registry with a focus on security and compliance.

**Key Features:**
- Image signing (Docker Content Trust)
- Built-in vulnerability scanning
- Role-based access control
- Image promotion policies
- Caching and mirroring
- Integration with Mirantis Kubernetes Engine

**Best For:** Enterprise teams with strict security and compliance requirements.

---

### Sonatype Nexus Repository

- **Website**: <https://www.sonatype.com/products/sonatype-nexus-repository>
- **Pricing**: Free OSS edition; paid Pro edition

A universal repository manager supporting Docker alongside many other package formats (Maven, npm, PyPI, etc.).

**Key Features:**
- Multi-format support (Docker, Helm, npm, Maven, PyPI, apt, yum, etc.)
- Proxy/cache remote registries
- Hosted and group repositories
- LDAP/SAML authentication
- Cleanup policies
- High availability (Pro)
- Vulnerability scanning via Nexus Lifecycle (Pro)

**Best For:** Teams managing multiple artifact types who want a single repository manager.

---

## Comparison Matrix

| Registry | License | OCI Compliant | Vuln Scanning | RBAC | Helm Support | UI | Replication |
|---|---|---|---|---|---|---|---|
| Docker Registry | Apache 2.0 | Yes | No | Basic | No | No | No |
| Harbor | Apache 2.0 | Yes | Yes (Trivy) | Yes | Yes | Yes | Yes |
| Portus | Apache 2.0 | Partial | No | Yes | No | Yes | No |
| Gitea | MIT | Yes | No | Yes | Yes | Yes | No |
| GitLab CR | MIT | Yes | Ultimate | Yes | Yes | Yes | Yes |
| Zot | Apache 2.0 | Yes | Yes | Yes | Yes | Yes | Yes |
| Project Quay | Apache 2.0 | Yes | Yes (Clair) | Yes | Yes | Yes | Yes |
| JFrog CR | Freemium | Yes | Paid | Yes | Yes | Yes | Yes |
| Nexus | Freemium | Yes | Paid | Yes | Yes | Yes | Paid |

---

## References

- [Harbor](https://goharbor.io/)
- [Distribution / Docker Registry](https://github.com/distribution/distribution)
- [Portus](http://port.us.org/)
- [Gitea](https://gitea.io/)
- [GitLab Container Registry](https://docs.gitlab.com/ee/user/packages/container_registry/)
- [Distr](https://distr.sh/)
- [Zot Registry](https://zotregistry.dev/)
- [Dragonfly](https://d7y.io/)
- [Project Quay](https://www.projectquay.io/)
- [JFrog Container Registry](https://jfrog.com/container-registry/)
- [Mirantis Secure Registry](https://www.mirantis.com/)
- [Sonatype Nexus Repository](https://www.sonatype.com/products/sonatype-nexus-repository)
