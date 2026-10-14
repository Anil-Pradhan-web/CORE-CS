# Dell Interview Revision Notes: API Specifications, Testing & Debugging

**Role:** Undergraduate Intern — Software Engineering  
**Goal:** Understand the basics and explain them clearly in an interview.

> Easy summary: define the expected behaviour first, build the API, then test whether the real behaviour matches the definition.

---

## 1. The Big Picture

Imagine we need to build a REST API to create and retrieve tasks. Before writing code, the team should agree on:

- Which endpoints are available?
- Which HTTP methods do they use?
- What data must the client send?
- What response will the server return?
- What happens if input is invalid or a task does not exist?
- How will we test that the API behaves as agreed?

A **specification** is a clear, agreed description of how a system or API is supposed to behave. It reduces confusion between developers, testers, and users.

### A simple workflow

1. Understand the requirement.
2. Write the API contract/specification.
3. Review and validate the specification.
4. Implement the API.
5. Test the implementation against the specification.
6. Fix mismatches and keep the documentation updated.

**Interview line:** “I would define the expected API behaviour first, review it with the team, implement it, and then run tests to verify that the actual behaviour matches the specification.”

---

## 2. REST API Basics

A REST API lets software systems communicate over HTTP.

### Important terms

- **Endpoint:** A URL path for a resource, such as `/tasks`.
- **HTTP method:** The action requested by the client.
- **Request:** Data sent by the client to the server.
- **Response:** Data returned by the server.
- **Status code:** A number describing the result.
- **Contract:** The agreed rules for requests, responses, and behaviour.

### Common HTTP methods

| Method | Typical purpose | Example |
|---|---|---|
| `GET` | Read data | `GET /tasks/12` |
| `POST` | Create a resource | `POST /tasks` |
| `PUT` | Replace/update a resource | `PUT /tasks/12` |
| `PATCH` | Partially update a resource | `PATCH /tasks/12` |
| `DELETE` | Delete a resource | `DELETE /tasks/12` |

### Common status codes

| Code | Meaning | Example |
|---|---|---|
| `200 OK` | Request succeeded | Task details returned |
| `201 Created` | A resource was created | New task created |
| `400 Bad Request` | Request is invalid | Malformed request |
| `401 Unauthorized` | Authentication is missing or invalid | Invalid/missing token |
| `403 Forbidden` | Caller is not allowed to do this | User lacks permission |
| `404 Not Found` | Resource does not exist | Task ID does not exist |
| `422 Unprocessable Content` | Request data failed validation in APIs that use this code | Missing/invalid field |
| `500 Internal Server Error` | Unexpected server error | Unhandled backend failure |

Exact status-code choices should be documented in the API contract and used consistently.

---

## 3. OpenAPI: Describe the API Contract

### What is OpenAPI?

**OpenAPI is a standard format for describing an HTTP API in a machine-readable way.** A specification often uses YAML or JSON.

It can describe API paths and methods, path/query parameters, request body structure, response structure and status codes, and authentication requirements. Tools can use an OpenAPI document to generate API documentation, client code, and some types of tests.

### Example: Create a task

A simple API contract could say:

- Method and path: `POST /tasks`
- Request body: `title` is required and must be a string.
- Success response: `201 Created`
- Error response: `400 Bad Request` or another documented validation error for invalid input.

A simplified OpenAPI example:

```yaml
openapi: 3.0.3
info:
  title: Task API
  version: 1.0.0
paths:
  /tasks:
    post:
      summary: Create a task
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required:
                - title
              properties:
                title:
                  type: string
                  minLength: 1
      responses:
        '201':
          description: Task created
          content:
            application/json:
              schema:
                type: object
                required:
                  - id
                  - title
                properties:
                  id:
                    type: integer
                  title:
                    type: string
        '400':
          description: Invalid request
```

This example is deliberately small. A real specification should also document all relevant parameters, validation rules, security, and error responses.

### Why define the contract first?

- The developer and tester know what behaviour is expected.
- Frontend and backend teams can work against the same agreement.
- Ambiguous requirements can be found before implementation.
- Automated tools can generate documentation or check the contract.

### Interview answer: “How would you specify a REST API before implementing it?”

> “First, I would clarify the requirement and identify the resources and operations. Then I would define the endpoints, HTTP methods, request parameters and body, response schemas, status codes, and authentication rules in an OpenAPI document. I would review it with the team and resolve unclear cases before coding. After implementation, I would test the API against this contract.”

---

## 4. JSON Schema: Define the Shape of JSON Data

### What is JSON Schema?

**JSON Schema describes and validates the structure and rules of JSON data.** For example, it can specify which fields are required, what type each field must have, and allowed value ranges.

It can be used for request data, response data, configuration files, and other JSON documents. In OpenAPI, schemas are commonly used to describe request and response bodies.

### Example

Suppose a task must have a title and may have a priority.

```json
{
  "type": "object",
  "required": ["title"],
  "properties": {
    "title": {
      "type": "string",
      "minLength": 1
    },
    "priority": {
      "type": "integer",
      "minimum": 1,
      "maximum": 5
    }
  },
  "additionalProperties": false
}
```

This rule says:

- The data must be a JSON object.
- `title` is required and must be a non-empty string.
- `priority` is optional, but if provided it must be an integer from 1 to 5.
- Extra fields are not allowed in this example.

Valid example:

```json
{"title": "Fix login bug", "priority": 3}
```

Invalid examples:

```json
{"priority": 3}
```

Invalid because `title` is missing.

```json
{"title": "Fix login bug", "priority": 9}
```

Invalid because priority is greater than 5.

### Interview answer: “What is JSON Schema, and how is it different from OpenAPI?”

> “JSON Schema defines the structure and validation rules for JSON data. OpenAPI describes the API itself, including endpoints, HTTP methods, parameters, responses, and security. OpenAPI can use schemas to describe the request and response data.”

---
## 5. BDD and Gherkin: Describe Behaviour with Examples

### What is BDD?

**BDD means Behaviour-Driven Development.** It encourages the team to describe expected software behaviour using clear examples that product, development, and testing teams can understand.

### What is Gherkin?

Gherkin is a readable format commonly used to write BDD scenarios. It uses keywords such as:

- **Given:** The starting situation or context.
- **When:** The action that happens.
- **Then:** The expected result.
- **And / But:** Additional conditions or results.

### Example: Create a task successfully

```gherkin
Feature: Create a task

  Scenario: Create a task with a valid title
    Given the task API is available
    When I send a POST request to "/tasks" with title "Fix login bug"
    Then the response status should be 201
    And the response should contain the task ID
    And the response title should be "Fix login bug"
```

### Example: Reject an invalid task

```gherkin
Scenario: Reject a task without a title
  Given the task API is available
  When I send a POST request to "/tasks" without a title
  Then the API should return the documented validation error
  And the task should not be created
```

These scenarios express what the API should do without tying the requirement to a particular implementation.

### Interview answer: “What is Given–When–Then?”

> “Given describes the starting context, When describes the action, and Then describes the expected outcome. It helps us turn requirements into clear, testable examples that developers and testers can agree on.”

---

## 6. Conformance Testing: Does the Code Follow the Specification?

### What does conformance mean?

**Conformance testing checks whether the real implementation follows its documented contract.**

For an API, we can check whether:

- The endpoint and HTTP method are correct.
- Required fields and data types are enforced.
- The response has the expected structure.
- The correct status code is returned.
- Authentication and permission rules are followed.
- Invalid input and missing resources produce documented errors.

### Example

The OpenAPI document says `POST /tasks` should return `201 Created` and a JSON response containing an integer `id` and a string `title`.

If the code returns `200 OK`, omits `id`, or returns `id` as an unexpected string, the implementation may not conform to the contract.

### How can we test conformance?

1. **Positive tests:** Send valid requests and check expected responses.
2. **Negative tests:** Send missing, malformed, or invalid data and check documented errors.
3. **Schema validation:** Check that requests and responses match their defined schemas.
4. **Contract/API tests:** Compare actual API behaviour with the OpenAPI contract.
5. **Integration tests:** Check that connected components work together correctly.
6. **Regression tests:** Re-run important tests after a code change to ensure previously working behaviour has not broken.

The exact tools depend on the project. Tests may be written directly or generated with suitable tools.

### Interview answer: “How would you verify that an API conforms to its specification?”

> “I would create positive and negative tests from the contract. I would verify the endpoint, status code, response fields and data types, required-field validation, authentication, and error cases. Where possible, I would use automated schema or contract tests to compare actual responses with the OpenAPI specification. I would run these tests in CI so that contract-breaking changes are caught early.”

---

## 7. Root-Cause Analysis: Find the Real Reason for a Failure

### What is root-cause analysis?

**Root-cause analysis means finding the underlying reason an issue happened, rather than only fixing the visible symptom.**

### Example

The API is expected to return `201 Created`, but a test receives `500 Internal Server Error`.

A weak response would be to change the test or hide the error. A better response is to investigate why the server returned 500.

### A practical debugging process

1. **Reproduce the issue:** Use the same request and conditions.
2. **Read the failure carefully:** Check the response, logs, stack trace, and test output.
3. **Trace the request:** Follow it through validation, business logic, database calls, and external services.
4. **Compare code with the specification:** Check what was expected versus what actually happened.
5. **Find the root cause:** For example, a missing database value, incorrect assumption, unhandled exception, or unclear requirement.
6. **Fix the cause:** Avoid adding a workaround that hides the real issue.
7. **Add a regression test:** Confirm the bug is fixed and does not return.
8. **Update the specification or documentation if needed:** The code and contract must agree.

### Interview answer: “How would you diagnose a mismatch between the implementation and the specification?”

> “First, I would reproduce the issue and compare the expected behaviour with the actual result. I would inspect logs, error messages, and the request flow to locate where the behaviour diverges. Then I would determine whether the cause is a code bug, invalid data, an integration problem, or an unclear specification. After fixing it, I would add a regression test and update the specification if the agreed behaviour itself needed to change.”

---

## 8. How These Concepts Fit Together

Use the same `POST /tasks` example:

1. **Requirement:** A user should be able to create a task with a title.
2. **OpenAPI:** Defines `POST /tasks`, its request, responses, and status codes.
3. **JSON Schema:** Defines the allowed structure and validation rules for the JSON body and response.
4. **Gherkin:** Describes examples of successful creation and invalid requests using Given–When–Then.
5. **Implementation:** The developer writes the API code.
6. **Conformance tests:** Tests check whether the code follows the API contract.
7. **Root-cause analysis:** If a test fails, the team investigates and fixes the actual reason.
8. **Living documentation:** The specification and documentation are kept in sync with the approved behaviour.

### Easy way to remember

- **OpenAPI = What is the API contract?**
- **JSON Schema = What shape must the JSON data have?**
- **Gherkin = What behaviour do we expect in a scenario?**
- **Conformance testing = Does the implementation follow the contract?**
- **Root-cause analysis = Why did it fail, and what is the real fix?**

---
## 9. Likely Interview Questions and Short Answers

### Q1. What is an API specification?

An API specification is a clear description of the API's endpoints, methods, inputs, outputs, status codes, and rules. It acts as an agreed contract between the API provider and its users.

### Q2. Why should we write the specification before the code?

It helps clarify requirements early, reduces misunderstanding, allows teams to work in parallel, and gives us a clear basis for testing.

### Q3. What is the difference between specification and implementation?

The specification describes what the system is expected to do. The implementation is the actual code that performs the work.

### Q4. What is the difference between testing and conformance testing?

Testing checks whether software behaves correctly for selected cases. Conformance testing specifically checks whether its behaviour follows a defined standard or contract, such as an API specification.

### Q5. What is a negative test?

A negative test checks how a system handles invalid input or an error condition, such as a missing required field or an unknown task ID.

### Q6. What does “specification is the source of truth” mean?

It means the approved specification is the agreed reference for expected behaviour. If the code and specification disagree, the team investigates the mismatch instead of assuming the code is correct. If requirements change, the specification should be reviewed and updated through the agreed process.

### Q7. What is a breaking API change?

A breaking change is a change that can cause existing API clients to stop working, such as removing a field they depend on or changing a response type unexpectedly. Such changes should be reviewed, tested, and managed carefully, often with a versioning or migration plan.

### Q8. What is CI, and why run tests in CI?

CI means Continuous Integration. It automatically builds and tests code changes. Running contract and regression tests in CI helps catch mismatches and breaking changes before they reach users.

### Q9. What would you do if a requirement is ambiguous?

I would ask the relevant stakeholders for clarification, write down the possible interpretations, agree on expected behaviour and edge cases, and update the specification before implementing it.

### Q10. What is living technical documentation?

It is documentation kept up to date as the approved system behaviour changes. Ideally, as much as possible is generated from or checked against the authoritative specification, reducing the risk that documentation describes behaviour the code does not have.

---

## 10. A Strong Combined Interview Answer

**Question:** “Suppose you are developing a REST API. How would you define its specification before implementation, and how would you verify that the implemented API conforms to that specification?”

**Sample answer:**

> “First, I would clarify the business requirement and identify the resources and operations the API needs. I would write an OpenAPI specification covering the endpoints, HTTP methods, parameters, request and response formats, status codes, and security requirements. I would define the JSON data structures and validation rules using schemas, and review edge cases with the team.
>
> I would also write behaviour scenarios in Given–When–Then format so that success and failure cases are clear. After the contract is reviewed, I would implement the API and create positive, negative, schema-validation, and contract tests. I would run these tests locally and in CI to check that the actual responses match the documented contract.
>
> If a mismatch appeared, I would reproduce it, inspect the logs and request flow, compare expected and actual behaviour, find the root cause, fix it, and add a regression test. If the agreed requirement changed, I would update the specification and its documentation as part of the review process.”

You do not need to memorise every word. Understand the order and explain it naturally.

---

## 11. Last-Minute Revision Checklist

Before the interview, make sure you can explain these in your own words:

- [ ] What an API specification is and why it comes before implementation.
- [ ] The difference between OpenAPI and JSON Schema.
- [ ] The meaning of Given, When, and Then in Gherkin.
- [ ] Positive tests vs negative tests.
- [ ] What conformance/contract testing checks.
- [ ] What a specification mismatch looks like.
- [ ] A basic root-cause analysis process.
- [ ] Why regression tests and CI are useful.
- [ ] What to do when a requirement is ambiguous.
- [ ] How to keep documentation aligned with approved behaviour.

**Final memory line:** Define the contract → implement it → test conformance → investigate failures → fix root causes → keep documentation aligned.
