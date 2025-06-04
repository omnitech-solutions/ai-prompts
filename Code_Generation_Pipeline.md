# Modular AI-Assisted Code-Generation Prompt Pipeline

---

## 1. 🧠 Information-Gathering Phase (Initial Understanding)

Collect comprehensive developer inputs about the task:

- **Task Definition & Scope:**  
  Specify the precise feature/module/app to build (e.g., "User authentication API with Okta SSO", "Frontend data grid with server-side filtering").

- **Domain Context & Background:**  
  Detail business logic, relevant architecture, tech stack versions (Rails 7, Laravel 11, React 18), existing codebases, compliance/security policies (e.g., OWASP Top 10).

- **Target Audience & Usage:**  
  Identify end users or stakeholders (engineers, product managers, customers), plus required tone/style for docs and comments.

- **Functional & Non-Functional Requirements:**  
  Specify input/output data, validation rules, API endpoints, UI components, performance targets, and security constraints.

- **Testing & Quality Expectations:**  
  Identify preferred test frameworks (RSpec, Pest), testing types (unit, integration, E2E with Cypress), coverage targets, and automation needs.

---

## 2. 🧩 Core Pipeline Components (Modular Code-Gen Segments)

### 2.1 Code Core

- **Models:**  
  Define domain entities, attributes, relations, validations, and persistence logic.

- **Services:**  
  Implement business rules, orchestrate domain logic, and handle validations (incl. reusable rule sets).

### 2.2 API Layer

- **Context Use Cases:**  
  Define explicit use case classes for request handling.

- **Endpoints:**  
  Map HTTP routes to use cases with clear input and output contracts.

- **Swagger/OpenAPI Docs:**  
  Auto-generate or define detailed API specification with request/response schemas and descriptions.

### 2.3 Frontend View

- **Vue/React Components:**  
  Build UI components (e.g., data grids, forms) using JSON-schema driven rendering for flexibility.

- **UI State & Validation:**  
  Implement client-side validation aligned with backend rules.

- **Dynamic Updates:**  
  Support schema-driven dynamic UI changes with clear mappings between frontend and backend data.

---

## 3. 🔄 Iterative Development & Testing Workflow

- **Simplistic Validation → Backend Rules:**  
  Start with minimal validation on frontend, progressively push complex logic to backend services.

- **Test-Driven Development:**
    - Write initial public test cases reflecting business rules.
    - Generate additional edge-case AI tests with double validation pass.
    - Run test-fix cycles iteratively until all tests pass.

- **Error Handling & Logging:**  
  Capture validation exceptions and aggregate errors in application context for reporting.

---

## 4. 📋 Output & Documentation Standards

- **YAML/JSON Structured Outputs:**  
  Standardise output with labelled sections (models, services, APIs, frontend) for easy review and automation.

- **Comments & Documentation:**  
  Include concise, contextual comments linking business logic to implementation and tests.

- **Tech Spec Generation:**  
  Produce architecture diagrams and BDD-style scenarios automatically from the specification.

---

## 5. 🎯 Final Deliverables & Recommendations

- **Complete Modular Code:**  
  Separate layers with clear boundaries (model, service, API, frontend).

- **Comprehensive Test Suite:**  
  Cover happy paths, edge cases, and failure modes.

- **Iteration Notes & Fix Logs:**  
  Document changes, rationale, and improvements during development cycles.

- **Future Enhancements:**  
  Suggest optimisation, refactoring, or scalability improvements.

---

# Example YAML Skeleton of Pipeline Output

```yaml
code_core:
  models:
    - name: "User"
      attributes: ["id", "email", "password_digest"]
      validations: ["presence", "uniqueness", "format"]
  services:
    - name: "UserRegistration"
      rules: ["validate_email", "password_strength"]
      logic_summary: "Creates user, sends confirmation email"

api_layer:
  use_cases:
    - name: "RegisterUser"
      inputs: ["email", "password"]
      outputs: ["success", "error_messages"]
  endpoints:
    - path: "/api/v1/users"
      method: "POST"
      use_case: "RegisterUser"
  swagger:
    doc_url: "/docs/api/v1/users"

frontend_view:
  components:
    - name: "UserRegistrationForm"
      schema_source: "UserRegistration"
      validation: "matches backend rules"
      dynamic_fields: true

testing:
  public_tests:
    - name: "Valid user registration"
      input: {...}
      expected: {...}
  ai_generated_tests:
    - name: "Reject weak passwords"
      input: {...}
      expected: {...}
  test_results:
    - test_name: "Valid user registration"
      status: "pass"
    - test_name: "Reject weak passwords"
      status: "fail"
      error: "Password too weak"

iteration_logs:
  - step: "Refined password regex to include special characters"
  - step: "Added frontend input sanitisation"
```

---

Would you like me to generate a detailed prompt example with this pipeline tailored to a specific project or feature you want to develop?
