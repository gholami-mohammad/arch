# General user stories sequence diagrams

-   Frontend: Control Panel UI
-   Auth Service
-   Control Panel Service
-   Mother Service
-   Test Service
-   Reporting Service
-   WebSocket Gateway
-   Notification/Toast

---

## 1. Control panel usable on desktop and mobile

```mermaid
sequenceDiagram
    actor Tester
    participant Browser
    participant Frontend as Control Panel (Responsive SPA)
    participant NextJs as NextJs server

    Tester->>Browser: Open control panel URL
    Browser->>Frontend: Open control panel on desktop/mobile
    Frontend->>NextJs: Fetch assets (JS/CSS)
    NextJs-->>Frontend: Serve responsive bundle
    Frontend-->>Browser: Render responsive UI
    Browser-->>Tester: View Page
```

---

## 2. User login

```mermaid
sequenceDiagram
    actor Tester
    participant Frontend as Control Panel UI
    participant Auth as Auth Service
    participant UserDB

    Tester->>Frontend: Enter email + password, click Login
    Frontend->>Auth: POST /login {email, password}
    Auth->>UserDB: Validate credentials
    UserDB-->>Auth: Valid / Invalid
    alt Valid credentials & password policy OK
    Auth-->>Frontend: 200 {sessionToken, userRole}
    Frontend-->>Tester: Redirect to dashboard
    else Invalid credentials or policy violation
    Auth-->>Frontend: 401/422 {error}
    Frontend->>Tester: Show error message
    end
```

---

## 3. User logout

```mermaid
sequenceDiagram
    actor Tester
    participant Frontend as Control Panel UI
    participant Auth as Auth Service
    participant WebSocket as WebSocket Gateway

    Tester->>Frontend: Click Logout
    Frontend->>Auth: POST /logout {sessionToken}
    Auth-->>Frontend: 200 OK (session terminated)
    Frontend->>WebSocket: Close real-time subscriptions
    Frontend-->>Tester: Redirect to Login
    Note over Frontend: Block access to protected routes & history back
```

---

## 4. Live view of mother service transactions

```mermaid
sequenceDiagram
  actor User
  participant Frontend as Control Panel UI
  participant WebSocket as Live Feed WebSocket Gateway
  participant Mother as Mother Service

  User->>Frontend: Open mother service live panel
  Frontend->>WebSocket: Subscribe to mother:{serviceId} stream
  WebSocket->>Mother: Register subscription
  loop Real-time updates
    Mother-->>WebSocket: Tx event {id, status, latency, errorRate}
    WebSocket-->>Frontend: Push event
    Frontend-->>User: Auto-update UI (no manual refresh)
  end
```

---

## 5. Live view of test service transactions

```mermaid
sequenceDiagram
  actor Tester
  participant Frontend as Control Panel UI
  participant WebSocket as Live Feed WebSocket Gateway
  participant Test as Test Service

  Tester->>Frontend: Open test service live panel
  Frontend->>WebSocket: Subscribe to test:{serviceId} stream
  WebSocket->>Test: Register subscription
  loop Real-time updates
    Test-->>WebSocket: Tx event {id, status, duration, scenarioId}
    WebSocket-->>Frontend: Push event
    Frontend-->>Tester: Auto-update UI
  end
```

---

## 6. Search in live panel (scenarios list)

```mermaid
sequenceDiagram
    actor Tester
    participant Frontend as Control Panel UI
    participant WebSocket as Live Feed WebSocket Gateway

    Tester->>Frontend: Type search query (test service, mother, scenario name)
    Frontend->>WebSocket: Subscribe on filtered websockets scenarios, tests, mother service
    WebSocket-->>Frontend: Event data
    Frontend-->>Tester: Render filtered list
```

---

## 8. Sort scenarios in live panel

```mermaid
sequenceDiagram
  actor Tester
  participant Frontend as Control Panel UI
  participant ControlPanel as Control Panel Service

  Tester->>Frontend: Set sort by time asc/desc
  Frontend->>ControlPanel: GET /scenarios?sort=time&order=asc|desc
  ControlPanel-->>Frontend: 200 {items sorted}
  Frontend-->>Tester: Render sorted list
```

---

# QA manager user stories sequence diagrams

## 9. Create new user

```mermaid
sequenceDiagram
  actor QA_Manager as QA Manager
  participant Frontend as Control Panel UI
  participant Auth as Auth/User Service
  participant Notification as Toast

  QA_Manager->>Frontend: Open create user form
  Frontend-->>QA_Manager: Show fields {firstName,lastName,email,status,role}
  QA_Manager->>Frontend: Submit form
  Frontend->>Auth: POST /users {data}
  alt Email unique & required fields OK
    Auth-->>Frontend: 201 {userId}
    Frontend->>Notification: Show success
    Frontend-->>QA_Manager: Update users list
  else Validation/duplication error
    Auth-->>Frontend: 422 {errors}
    Frontend->>Notification: Show validation errors
  end
```

---

## 10. View users list

```mermaid
sequenceDiagram
  actor QA_Manager
  participant Frontend as Control Panel UI
  participant Auth as Auth/User Service
  participant Notification as Toast

  QA_Manager->>Frontend: Open users list
  Frontend->>Auth: GET /users
  alt Data available
    Auth-->>Frontend: 200 [{name,email,status,role,createdAt}]
    Frontend-->>QA_Manager: Render list
  else Empty or error
    Auth-->>Frontend: 4xx or 5xx
    Frontend->>Notification: Show error
  end
```

---

## 11. Edit user info

```mermaid
sequenceDiagram
  actor QA_Manager
  participant Frontend as Control Panel UI
  participant Auth as Auth/User Service
  participant Notification as Toast

  QA_Manager->>Frontend: Select user to edit
  Frontend->>Auth: GET /users/{id}
  Auth-->>Frontend: 200 {user data}
  Frontend-->>QA_Manager: Show edit form (email read-only)
  QA_Manager->>Frontend: Save changes {name,lastName,status,role}
  Frontend->>Auth: PUT /users/{id} {payload}
  alt Save OK
    Auth-->>Frontend: 200 {updated}
    Frontend->>Notification: Show success
    Frontend-->>QA_Manager: Refresh list
  else Error
    Auth-->>Frontend: 4xx/5xx {error}
    Frontend->>Notification: Show error
  end
```

---

## 12. Deactivate user

```mermaid
sequenceDiagram
    actor QA_Manager
    participant Frontend as Control Panel UI
    participant Auth as Auth/User Service
    participant Notification as Confirm/Toast

    QA_Manager->>Frontend: Click Deactivate
    Frontend->>Notification: Show confirmation modal
    QA_Manager->>Frontend: Confirm
    Frontend->>Auth: PATCH /users/{id} {status:inactive}
    alt Save OK
        Auth-->>Frontend: 200 {status:inactive}
        Frontend->>Notification: Show success
        Frontend-->>QA_Manager: Refresh list
    else Error
        Auth-->>Frontend: 4xx/5xx {error}
        Frontend->>Notification: Show error
    end
    Note over Auth: Block login for inactive users
```

---

## 13. Soft-delete user

```mermaid
sequenceDiagram
    actor QA_Manager
    participant Frontend as Control Panel UI
    participant Auth as Auth/User Service
    participant Notification as Confirm/Toast

    QA_Manager->>Frontend: Click Delete
    Frontend->>Notification: Show warning confirmation
    QA_Manager->>Frontend: Confirm delete
    Frontend->>Auth: DELETE /users/{id} (soft delete)
    alt Save OK
        Auth-->>Frontend: 200 {deleted:true}
        Frontend->>Notification: Show success
        Frontend-->>QA_Manager: Refresh list
    else Error
        Auth-->>Frontend: 4xx/5xx {error}
        Frontend->>Notification: Show error
    end
```

---

# QA manager and Tester user stories sequence diagrams

## 14. Reports (manager)

```mermaid
sequenceDiagram
  actor Tester as QA Manager or Tester
  participant Frontend as Control Panel UI
  participant Reporting as Control Panel Service
  participant Notification as Toast

  Tester->>Frontend: Open scenario results page
  Frontend->>Reporting: GET /reports/scenarios?page=...&size=...
  alt Data available
    Reporting-->>Frontend: 200 [{runId,scenarioId,name,status,txTotal,txFailed,startTime,duration}]
    Frontend-->>Tester: Render paginated table, sort enabled
  else error
    Reporting-->>Frontend: 4xx or 5xx
    Frontend->>Notification: Show error
  end
```

---

## 15. Filter and search in reports (manager)

```mermaid
sequenceDiagram
  actor Tester as QA Manager or Tester
  participant Frontend as Control Panel UI
  participant Reporting as Control Panel Service

  Tester->>Frontend: Apply filters {timeRange,status,nameOrId}
  Frontend->>Reporting: GET /reports/scenarios?filters=...
  Reporting-->>Frontend: 200 {filtered results}
  Frontend-->>Tester: Update table without page reload
```

---

## 16. Export reports (manager)

```mermaid
sequenceDiagram
  actor Tester as QA Manager or Tester
  participant Frontend as Control Panel UI
  participant Reporting as Control Panel Service
  participant Notification as Toast

  Tester->>Frontend: Click Export (CSV/JSON)
  Frontend->>Reporting: POST /reports/export {filters, format}
  alt Normal dataset
    Reporting-->>Frontend: 200 {downloadUrl}
    Frontend-->>Tester: Trigger download
  else Large dataset (queued)
    Reporting-->>Frontend: 202 {jobId, eta}
    Frontend->>Notification: Show queued/export time estimate
  end
```

---

## 17. Start scenario

```mermaid
sequenceDiagram
  actor Tester as QA Manager or Tester
  participant Frontend as Control Panel UI
  participant Scenario as Control Panel Service
  participant WebSocket as WebSocket Gateway

  Tester->>Frontend: Click Start on scenario
  Frontend->>Scenario: POST /scenarios/{id}/start
  alt Pre-checks pass (single mother, not running)
    Scenario-->>Frontend: 200 {status:running}
    Frontend->>WebSocket: Subscribe to scenario:{id} updates
    Frontend-->>Tester: Show status "Running"
  else Already running / missing mother
    Scenario-->>Frontend: 409/422 {error}
    Frontend-->>Tester: Show error
  end
```

---

## 18. Pause scenario

```mermaid
sequenceDiagram
  actor Tester as QA Manager or Tester
  participant Frontend as Control Panel UI
  participant Scenario as Control Panel Service
  participant WebSocket as WebSocket Gateway

  Tester->>Frontend: Click Pause
  Frontend->>Scenario: POST /scenarios/{id}/pause
  alt Available to pause
    Scenario-->>Frontend: 200 {status:paused}
    WebSocket-->>Frontend: Status update
    Frontend-->>Tester: Show "Paused"
  else Already paused
    Scenario-->>Frontend: 409 {error}
    Frontend-->>Tester: Show error
  end
```

---

## 19. Restart scenario

```mermaid
sequenceDiagram
  actor Tester as QA Manager or Tester
  participant Frontend as Control Panel UI
  participant Scenario as Control Panel Service
  participant WebSocket as WebSocket Gateway

  Tester->>Frontend: Click Restart
  Frontend->>Scenario: POST /scenarios/{id}/restart
  Scenario-->>Frontend: 200 {status:running}
  WebSocket-->>Frontend: Status update
  Frontend-->>Tester: Show "Running"
```

---

## 20. Abort scenario

```mermaid
sequenceDiagram
  actor Tester as QA Manager or Tester
  participant Frontend as Control Panel UI
  participant Scenario as Control Panel Service
  participant WebSocket as WebSocket Gateway
  participant Notification as Confirm/Toast

  Tester->>Frontend: Click Abort
  Frontend->>Notification: Show confirm
  Tester->>Frontend: Confirm abort
  Frontend->>Scenario: POST /scenarios/{id}/abort
  Scenario-->>Frontend: 200 {status:aborted}
  WebSocket-->>Frontend: Status update
  Frontend-->>Tester: Remove/mark scenario as aborted
```

---

## 21. Define mother/test services (Shared between starting mother service and test service)

### Part1

```mermaid
sequenceDiagram
  actor Tester as QA Manager or Tester
  participant Frontend as Control Panel UI
  participant ControlPanelHTTP as Control Panel Service
  participant Kafka as Kafka
  participant Database as PostgreSQL
  participant Notification as Toast

  Tester->>Frontend: Open the form
  Frontend-->>Tester: Show params
  Tester->>Frontend: Submit
  Frontend->>ControlPanelHTTP: POST /service-start {config}
  alt Valid config
    ControlPanelHTTP->>Database: Store service(s) as pending
    ControlPanelHTTP->>Kafka: Produce provisioning event
    ControlPanelHTTP-->>Frontend: 201 {id}
    Frontend->>Notification: Show success

  else Validation errors
    ControlPanelHTTP-->>Frontend: 422 {errors}
    Frontend->>Notification: Show errors
  end
  Frontend -->> Tester: Refresh list
```

### Part2: Event Consumer

```mermaid
sequenceDiagram
    participant Kafka as Kafka
    participant ControlPanelProvisioning as Control Panel Provisioning
    participant K8S as Kubernetes
    participant Database as PostgreSQL

    Kafka ->> ControlPanelProvisioning: Consume provisioning Event
    ControlPanelProvisioning ->> K8S: Start Mother/Test service

    alt Valid config
        K8S-->>ControlPanelProvisioning: Success
        ControlPanelProvisioning->>Database: Update status to provisioning
    else Validation errors
        ControlPanelProvisioning->>Database: Update status to failed
    end
```

### Part3: Background Job status check

```mermaid
sequenceDiagram
    participant ControlPanelProvisioningChecker as Control Panel Provisioning checker
    participant Database as PostgreSQL
    participant K8S as Kubernetes

    ControlPanelProvisioningChecker ->> Database : Load services with provisioning status
    ControlPanelProvisioningChecker ->> K8S: Check Mother/Test service running status

    alt Valid config
        K8S-->>ControlPanelProvisioningChecker: Success
        ControlPanelProvisioningChecker->>Database: Update status to provisioned
    else Validation errors
        ControlPanelProvisioningChecker->>Database: Update status to failed
    end
```

---

## 22. Mother/Test services list

```mermaid
sequenceDiagram
  actor Tester as QA Manager or Tester
  participant Frontend as Control Panel UI
  participant ControlPanelHTTP as Control Panel HTTP
  participant Database as PostgreSQL

  Tester->>Frontend: Open mother/test services list
  Frontend->>ControlPanelHTTP: GET /services
  ControlPanelHTTP->>Database: Load mother/test services
  Database-->>ControlPanelHTTP: Mother/test services
  ControlPanelHTTP-->>Frontend: 200 [{id,status,errorRate,delayPercent}]
  Frontend-->>Tester: Show list with start/pause/delete actions
```

---

## 23. Mother/test service details

```mermaid
sequenceDiagram
    actor Tester as QA Manager or Tester
    participant Frontend as Control Panel UI
    participant ControlPanelHTTP as Control Panel HTTP

    Tester->>Frontend: Open mother/test service details
    Frontend->>ControlPanelHTTP: GET /services/{id}
    ControlPanelHTTP->>Database: Load mother/test service
    Database-->>ControlPanelHTTP: Mother/test service details
    ControlPanelHTTP-->>Frontend: 200 {name,errorRate,delayPercentFixed,delayFixedMs,delayPercentRandom,delayMinMs,delayMaxMs}
    Frontend-->>Tester: Render detailed config
```

---

# Review Pending

## 26. Manage a specific test scenario

```mermaid
sequenceDiagram
  actor Tester as QA Manager or Tester
  participant Frontend as Control Panel UI
  participant Scenario as Control Panel Service
  participant Test as Test Service

  Tester->>Frontend: Open scenario management
  Frontend->>Scenario: GET /scenarios/{id}
  Scenario-->>Frontend: 200 {status,mother,tests,params}
  Frontend-->>Tester: Show tests with status, error/delay params
  Tester->>Frontend: Add test service / start / pause / delete
  Frontend->>Scenario: POST/PATCH/DELETE actions
  Scenario-->>Frontend: 200 {updated}
  Frontend-->>Tester: Reflect changes
```

---

## 27. View all test scenarios

```mermaid
sequenceDiagram
  actor Tester as QA Manager or Tester
  participant Frontend as Control Panel UI
  participant Scenario as Control Panel Service

  Tester->>Frontend: Open scenarios list
  Frontend->>Scenario: GET /scenarios
  Scenario-->>Frontend: 200 [{id,status,motherName,...}]
  Frontend-->>Tester: Render all scenarios with statuses and mother linkage
```

---
