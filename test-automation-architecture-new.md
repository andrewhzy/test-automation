# Test Automation Service Architecture

## Overview
This document describes the architecture for an automated testing service that tests Glean chat API using multiple user accounts and predefined questions.

## Requirements
- around 20 test users for automated testing
- around 4 authorized users who can trigger tests
- around 30 predefined questions for testing
- Automated collection of responses in Excel format
- Configurable users and questions

## High-Level Architecture

```mermaid
graph TB
    Client[Client<br/>Web UI/CLI/Scripts]
    SSO[SSO Service<br/>Authentication]
    TestAuto[Test-Auto Service<br/>Main Service]
    Glean[Glean Service<br/>Chat API]
    
    Client -->|<1>. Authenticate| SSO
    SSO -->|<2>. SSO token| Client
    Client -->|<3>. Test Request| TestAuto
    TestAuto -->|"<4>. Chat Requests<br/>(as test users)"| Glean
    Glean -->|<5>. Responses| TestAuto
    TestAuto -->|"<6>. ResultSet"| Client
```

## Component Details

### Client
- Authenticate test operator and get SSO token
- Call Test-Auto Service to perform tests

### SSO Service
- Handle authentication and return SSO token

### Test-Auto Service
Main orchestration service with the following modules

### Glean Service
- Target service being tested
- Provides chat API and authentication endpoints

## Authentication & Authorization Flow

```mermaid
sequenceDiagram
    participant C as Client
    participant SSO as SSO Service
    participant TA as Test-Auto Service
    participant G as Glean Service
    
    C->>SSO: Authenticate Test Operator
    SSO->>C: SSO Token
    C->>TA: Test Request (with SSO token, users, questions)
    TA->>TA: Validate SSO Token
    
    alt Authorization Checks
        TA->>TA: Check operator in allowlist
        TA->>TA: Check test users in allowlist
        TA->>TA: Check questions and application_ids are in allowlist
    else Authorization Failed
        TA->>C: Error: Unauthorized
    end
    
    loop For each test user
        TA->>G: POST /rest/api/v1/createauthtoken (with service token, impersonate user)
        G->>TA: User auth token
        loop For each question
            TA->>G: POST /rest/api/v1/chat (with user token, question)
            G->>TA: Chat response
            TA->>TA: Store result with metadata
        end
    end

    TA->>C: Response with resultset
```

## Data Structures

### Request Format
```json
{
  "user": "user1",
  "questions": [
    "What is machine learning?",
    "What is machine MCP?",
    "What is machine Agent?",
    "How does neural network training work?"
  ]
}
```

### Response Format
```json
{
  "test_id": "uuid-12345",
  "timestamp": "2024-01-01T00:00:00Z",
  "requested_by": "operator1",
  "test_user_id": "testuser1",
  "status": "completed",
  "results": [
    {
      "question": "What is machine learning?",
      "answer": "Machine learning is a subset of artificial intelligence...",
      "citation_urls": [
        "https://internal-docs.com/ml-overview",
        "https://wiki.company.com/ai-basics"
      ],
      "response_time_ms": 1500,
      "timestamp": "2024-01-01T00:00:01Z",
      "status": "success"
    },
  ]
}
```

## Configuration Management

### Test Operator Allowlist
```yaml
test_operators:
  - operator1
  - operator2
  - operator3
  - operator4
```

### Test Users and Questions Allowlist
```yaml
test_auto_config:
  testuser1:
    questions:
      - "What is machine learning?"
      - "How does neural network training work?"
      - "What are the benefits of cloud computing?"
      - "How do I reset my password?"
      - "What is our company's mission statement?"
    application_ids:
      - "chat_ekb1"
      - "app2"
      - "app3"
  testuser2:
    questions:
      - "What is machine learning?"
      - "How does neural network training work?"
      - "What are the benefits of cloud computing?"
      - "How do I reset my password?"
      - "What is our company's mission statement?"
    application_ids:
      - "chat_ekb1"
      - "app2"
      - "app3"
  testuser3:
    questions:
      - "What is machine learning?"
      - "How does neural network training work?"
      - "What are the benefits of cloud computing?"
      - "How do I reset my password?"
      - "What is our company's mission statement?"
    application_ids:
      - "chat_ekb1"
      - "app2"
      - "app3"
  # ... up to 20 test users
```

## Security Considerations
- All API calls authenticated through SSO
- Multi-level authorization checks:
  - Test operator must be in allowlist
  - Test users must be in allowlist
  - Questions and application_ids must be pre-approved for each test user
- Service token for Glean API stored securely (environment variable/secrets manager)
- Audit logging of all test executions