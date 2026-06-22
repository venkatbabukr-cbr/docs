# LLD Guidelines for Cohort Management Platform

## Back-End Design (Cohort Platform) 

### DB Schema 
* Design for multi-tenancy
* Schema design matching OMOP schema
* Data isolation and secure storage per Organization/Cohort 
* DB versioning / migration – Alembic/Flyway scripts 

### Cloud Native & Scalability 
* Deployable on k8s 
* Production monitoring and observability 

### Design Patterns 
* Microservices arch 
* Modular monoliths 
* Multi-tenancy - Support easier onboarding and operations for any Cohort at any time. 
* Layered architecture 
* Interface/Implementation Abstraction with Plug and Play 
    * DB plug & play 
    * CMS plug & play 

### API Design 
See [Rest APIs Design Guide](REST_APIs_Design_Guidelines.md)

Few important points for consideration:

#### Security 
* JWT access token in all requests 
* Secure Front-end & Back-end communication 
* Secure M2M API access 
* Handle CSRF/XSRF 
* CORS 

#### Authentication & Authorization 
* RBAC/RABAC for user access. 
* Scope based authorization for M2M access. 

#### API Contracts 
* API versioning 
* API contract with detailed documentation and detailed request & response objects for consumer systems. 
* OpenAPI spec, Swagger docs, Redocs 

### Overall Security
* Security of data at rest
    * No PIIs to be stored
* Secure storage of Private keys/secrets etc... 

### Operations & Environments 
* Logging & easier debugging/troubleshooting 
* Configurable environments 
    * Dev/Staging/Prod 

---

## Front-End Design 

### API Integration 

### Security 
* Secure communication (End-to-End Encryption via HTTPS)
* Auth for all API calls 
* Secure storage of Cookies & Auth tokens 
* Gracefully handle auth expiry / Prefetch tokens 
* Handle CSRF/XSRF 
* XSS mitigation 
* CORS 

### Data & Error Handling
* Idempotency 
* Handle paginations 
* Graceful handling of API failures 

### SharePoint CMS 
* *(Should be handled from backend majorly)*
* Embedded viewers to show documents/files from sharepoint 

### Routes & States 
* For SPA – Build UI from the URL having paths and query parameters
    * Handle redirects from SSO/IAM etc... 

### UI/UX 
* Responsive UI 
* Configurable Themes 

### Operations & Environments 
* Configurable environments 
    * Dev/Staging/Prod 
* Logging 
    * Limit logging in Prod

---

## Quality assurance & Operations (Both backend and front-end)

* Test cases with coverage 
    * Unit-tests
    * Integration tests
    * E2E tests 
* Add linting & code quality metrics/analysis 
* PR process for review and integration of code changes 
* CI & CD 
    * Generate Test reports with Coverage 
    * Code quality reports 
    * Quality gates: Gated CI/CD pipelines 
* Production monitoring & observability 
* Documentation
