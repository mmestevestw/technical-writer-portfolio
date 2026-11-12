# Executive Product Summary: Order Care (OC) & Catalog Manager (CM) 22.10

## 1. Executive Summary & Product Roles

* **Order Care (OC 22.10):** A centralized order management and workflow orchestration engine designed to process, decompose, route, and track commercial and service orders across telecom systems, business units, and manual work activity groups.

* **Catalog Manager (CM 22.10):** A centralized, TM Forum SID-compliant catalog management system serving as the enterprise single point of truth for defining commercial product offerings, technical service specifications, and resource attributes across the product lifecycle.

* **Synergy & Dependency:** Order Care relies directly on Catalog Manager for product/service catalog definitions and rules. Order Care cannot operate standalone without an active Catalog Manager deployment.

## 2. Core Architecture & Architectural Layers

Both products share a hybrid platform layer and transition model (Application Virtual Machine / Platform Services).

### Order Care (OC)

```
+-----------------------------------------------------------------------+
|                       User Interface Applications                     |
|         (OM Dashboard, CPQ UI, Service Ordering Portal, SR Viewer)    |
+-----------------------------------------------------------------------+
|                            Optional Modules                           |
| (NgOM, CPQ, Service Ordering Portal, Process Controller, PM [Deprecated])|
+-----------------------------------------------------------------------+
|                               OC Baseline                             |
|          (Order Manager, Service Registry, Velocity Studio)          |
+-----------------------------------------------------------------------+
|                                Platform                               |
|        (Application Virtual Machine, Orchestration Framework, DBs)    |
+-----------------------------------------------------------------------+
```

### Catalog Manager (CM)

```
+-----------------------------------------------------------------------+
|                          Presentation / GUIs                          |
|             (Catalog Designer UI, Offer Manager UI, PLM UI)            |
+-----------------------------------------------------------------------+
|                             Design & Runtime                          |
|    (Commercial Browsing, Technical Browsing, Product/Service Mgmt)    |
+-----------------------------------------------------------------------+
|                          Functions & Modeling                         |
|   (Catalog Entity Modeling, Price Modeling, Rule Engine, MDR, PLM)   |
+-----------------------------------------------------------------------+
|                                Platform                               |
|       (Application Virtual Machine, Orchestration Framework, Kafka)   |
+-----------------------------------------------------------------------+
```

## 3. Key Features & Functional Modules

### Order Care (OC) Modules

* **Order Manager (OM):** Orchestrates the full order lifecycle using the Orchestration Framework (OF); handles order capture, decomposition, routing, mapping, fallout management, and exception escalation.

* **Next-Generation Order Manager (NgOM):** Cloud-native, high-throughput microservice evolution built on Spring Boot and Apache Cassandra capable of executing up to 1,500 orders/sec.

* **Configure Price Quote (CPQ):** Modular interface for customer/location search, catalog browsing, shopping cart/quote lifecycle management, digital signature integrations, and order submission.

* **Service Registry (SR):** SID-compliant subscription inventory holding sold offers, products, services, and resources with built-in versioning and integration with Catalog Manager.

* **Process Controller (PC):** Microservice for defining and executing BPMN 2.0 fulfillment processes via Camunda Engine, integrating with Order Manager and external OSS/BSS systems.

* **Service Ordering Portal (SOP):** Web portal for manual service order creation, bulk file importing, and tracking.

* **Velocity Studio:** Development platform for building, customizing, and deploying J2EE OSS/BSS enterprise solution modules.

### Catalog Manager (CM) Modules

* **Catalog Designer & New Catalog Designer:** Design-time tools to model custom entities, pricing algorithms, product offerings, rules, and specifications.

* **Offer Manager:** Angular-based GUI adding a commercial overlay over technical building blocks to assemble sellable market offers quickly.

* **Product Lifecycle Management (PLM) & PLM Manager:** Standardized BPMN 2.0 governance tool to drive product development from idea conception through active deployment to retirement.

* **Catalog Environments:** Separates **Design Time** (project-based creation, modeling, validation, and testing) from **Runtime** (read-only, high-performance operational catalog data access).

## 4. TMF Compliance & Standards

Order Care and Catalog Manager natively align with TM Forum Open APIs and the SID Information Model.

| Feature / Interface Domain | OC API Specification | CM API Specification | Notes / Purpose |
| :--- | :--- | :--- | :--- |
| **Product Catalog Management** | — | TMF620 / TMF620E | Commercial offer & spec management |
| **Product Ordering Management** | TMF622 (v4.0 / v5.0) | — | Commercial order placement & handling |
| **Service Catalog Management** | — | TMF633 (v3.0 / v4.0) | Technical & customer service specs |
| **Product Inventory Management** | TMF637 (v4.0) | — | Active product instances for customers |
| **Service Inventory Management** | TMF638 (v3.0 / v4.0) | — | Active service instance inventory |
| **Service Ordering** | TMF641 (v4.0) | — | Execution & provisioning of service orders |
| **Technical Browsing** | — | TMF645 (v4.0) | Real-time querying of service/resource specs |
| **Agreement Management** | TMF651 (v4.0) | — | Framework & subscription agreements |
| **Shopping Cart** | TMF663 (v4.0) | — | Cart validation, pricing, and checkout |
| **Commercial Browsing** | — | TMF679 (v4.0) | Runtime browsing of commercial catalog |

## 5. Platform Architecture, Upgrades & Ingress

### In-Service Software Upgrade (ISSU)

* Supports zero-downtime rolling upgrades on Kubernetes using Helm.

* Key Parameters: Minimum 2–3 replicas, `maxSurge` $\ge 1$, `maxUnavailable` $= 0$ (or minimal), fine-tuned `readinessProbe` (HTTPS `GET /pob/isAlive`), `terminationGracePeriodSeconds` (30–90s), and `PodDisruptionBudgets` (PDB).

* **ISSU Target KPI:** Accepts up to 5% potential traffic loss during controlled upgrades under standard load, with near-zero impact achievable with sufficient hardware headroom.

### Ingress Architecture (Contour / Envoy)

* **Standard Ingress:** Platform Ingress Controller (`tm-ingress-controller-cr`) using **Contour** (Control Plane) and **Envoy** (Data Plane) via dynamic xDS gRPC.

* **3-Layer HTTPProxy Hierarchy:**

  1. *Level 1 (Platform Root):* Defines virtualhost FQDN and TLS secrets.

  2. *Level 2 (Product Delegate):* Includes product delegation tables (e.g., `cm-services`, `oc-services`).

  3. *Level 3 (Service HTTPProxy):* Defines path prefixes, backend upstream TLS (`port 8443`), timeouts, and cookie-based session affinity.

* *Note:* Legacy NGINX Ingress is deprecated starting in release 22.9 and scheduled for removal in 22.11.

## 6. Licensing & Security Framework

### Licensing Lifecycle (Platform LM)

* **Centralized Server:** Operates via Platform License Manager (LM) fetching keys from the Network License Server (NeLS).

* **Resilience & Grace Period:** Local 24-hour cache allows running pods to continue unaffected if Platform LM connection fails. Since v21.0, expired licenses allow an infinite grace period for active pods until restart. If restarted without a valid license, startup fails.

* **Deprecation Notice:** Project Manager (PM) capacity peak licensing and PM UI are deprecated and scheduled for full removal.

### Security & User Management (Keycloak)

* **AVM Microservices:** Map Keycloak JWT `iam_roles` directly to AVM groups and privileges. Supports multitenancy using Keycloak `Clients (Tenant)` structures mapped to `resource_access` tokens.

* **Non-AVM Microservices:** Authentication via localized `AuthorisationProfile.json` maps or dynamic Keycloak role synchronization using Open Policy Agent (OPA) sidecar containers.