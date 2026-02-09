Content tracking management system
This illustrate the configserver deployment configuration for client services.

---

##  Service Configs

### **Global Defaults (`application.yml`)**
- Common actuator settings
- Eureka defaults
- JWT issuer (Keycloak)
- Logging levels

### **User Service (`userservice-k8s.yml`)**
- Runs on random port (`server.port: 0`)
- Connects to `userservicedb`
- Registers with Eureka
- Validates JWTs via Keycloak

### **Course Service (`courseservice-k8s.yml`)**
- Connects to `courseservicedb`
- Registers with Eureka
- JWT validation enabled

### **Notification Service (`notificationservice-k8s.yml`)**
- Connects to `notificationservicedb`
- Registers with Eureka
- JWT validation enabled

### **User Assessment Service (`userassessmentservice-k8s.yml`)**
- Connects to `userassessmentservicedb`
- Registers with Eureka
- JWT validation enabled

### **Analytics Service (`analyticsservice-k8s.yml`)**
- Connects to `analyticsservicedb`
- Registers with Eureka
- JWT validation enabled

### **API Gateway (`apigateway-k8s.yml`)**
- Runs on port `9091`
- Registers with Eureka
- Validates JWTs via Keycloak
- Defines routes to all microservices using `lb://{service}` URIs

### **Cloak Resource Server (`cloak-resource-server.yml`)**
- Runs on port `8080`
- Registers with Eureka
- Validates JWTs via Keycloak
- CORS allowed origins: `http://localhost:3000`, `http://localhost:4200`

### **Eureka Server (`eurekaserver.yml`)**
- Runs on port `8761`
- Acts as service registry
- Does not register itself
- Exposes health/info endpoints

---

##  Usage

1. **Config Server Setup**
   - Config Server points to this repo:
     ```yaml
     SPRING_CLOUD_CONFIG_SERVER_GIT_URI=https://github.com/simongawar/CMS-Project-config.git
     ```
   - Clients import config:
     ```yaml
     spring.config.import=optional:configserver:http://configserver:8071
     ```

2. **Profiles**
   - Each service sets:
     ```yaml
     SPRING_PROFILES_ACTIVE=k8s
     ```
   - Config Server serves the corresponding `*-k8s.yml`.

3. **Database & Secrets**
   - DB credentials and sensitive values are injected via Kubernetes Secrets (`cms-secrets`).
   - Config files reference them using `${DB_USERNAME}`, `${DB_PASSWORD}`, etc.

---

##  Notes
- **Do not commit real passwords or tokens** into this repo. Use placeholders and override via Kubernetes Secrets.
- Keep service names consistent with `spring.application.name` to avoid mismatches.
- Use `application.yml` for shared defaults, and service-specific files for overrides.

---

## Example Request Flow
1. `userservice` starts with `SPRING_PROFILES_ACTIVE=k8s`.
2. It contacts Config Server (`http://configserver:8071`).
3. Config Server serves `userservice-k8s.yml` + `application.yml`.
4. Service registers with Eureka and becomes discoverable by API Gateway.

---

