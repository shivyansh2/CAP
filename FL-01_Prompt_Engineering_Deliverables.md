# FL-01 Prompt Engineering Deliverables

---

## Part 1: Prompt Ladder Document

### Baseline (v0)
* **Prompt:** `Write backend code for user authentication`
* **Output Excerpt:**
  > "Here is a simple Node.js express user authentication example using bcrypt and JWT:
  > ```js
  > const express = require('express');
  > const bcrypt = require('bcrypt');
  > const jwt = require('jsonwebtoken');
  > const app = express();
  > app.use(express.json());
  > const users = []; // In-memory user store
  > app.post('/register', async (req, res) => { ... })
  > ```"

---

### Version 1: Adding Real Context
* **Prompt:** `Write backend code for user authentication. The project is an existing FastAPI (Python) application using PostgreSQL and SQLAlchemy.`
* **Output Excerpt:**
  > "Here is how to set up authentication in FastAPI using SQLAlchemy and Passlib:
  > ```python
  > from fastapi import FastAPI, Depends, HTTPException
  > from sqlalchemy.orm import Session
  > from passlib.context import CryptContext
  > pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")
  > # Generates standard OAuth2 password bearer flow...
  > ```"
* **Notes:**
  * **What changed in the prompt:** Added Real Context (tech stack: FastAPI, Python, PostgreSQL, SQLAlchemy).
  * **What improved in the output:** It stopped giving Node.js/Express code and generated Python code directly compatible with the target stack.
  * **What still failed:** The code is generic boilerplate using deprecated `passlib` patterns, has no security features like token refresh or hashing best practices, and lacks specific endpoint definitions.
  * **What to try next:** Add a clearer goal specifying production-ready OAuth2 JWT token architecture.

---

### Version 2: Adding a Clearer Goal
* **Prompt:** `Write backend code for user authentication. The project is an existing FastAPI (Python) application using PostgreSQL and SQLAlchemy. The goal is to implement production-grade JWT authentication with login, registration, and access/refresh token rotation.`
* **Output Excerpt:**
  > "Here is a multi-file setup for JWT authentication with refresh tokens in FastAPI... [Generates 150 lines of code including `/login`, `/register`, `/refresh` endpoints, and Pydantic schemas]"
* **Notes:**
  * **What changed in the prompt:** Added a Clearer Goal (JWT with access/refresh token rotation endpoints).
  * **What improved in the output:** The AI generated a complete flow including token rotation and refresh endpoints instead of a naive single-token dummy setup.
  * **What still failed:** **This addition actually made the output worse in usability.** The response dumped a massive 150-line monolith of combined Pydantic schemas, database models, and routes in a single code block without folder structures or separation of concerns.
  * **What to try next:** Add a specified output format to force modular file organization.

---

### Version 3: Adding a Specified Output Format
* **Prompt:** `Write backend code for user authentication. The project is an existing FastAPI (Python) application using PostgreSQL and SQLAlchemy. The goal is to implement production-grade JWT authentication with login, registration, and access/refresh token rotation. Format the output as modular files matching a standard FastAPI project structure (e.g., schemas.py, models.py, router.py, security.py).`
* **Output Excerpt:**
  > "### `app/schemas.py`
  > ```python
  > class UserCreate(BaseModel): ...
  > ```
  > ### `app/security.py`
  > ```python
  > def create_access_token(...): ...
  > ```
  > ### `app/routers/auth.py`
  > ```python
  > @router.post("/login") ...
  > ```"
* **Notes:**
  * **What changed in the prompt:** Added Specified Output Format (modular file separation).
  * **What improved in the output:** The response became immediately copy-paste ready, cleanly organized into file headers (`schemas.py`, `security.py`, etc.) instead of a monolithic wall of code.
  * **What still failed:** It used outdated password hashing (`passlib` which has unmaintained dependencies in modern Python environments) and omitted critical security protections like HTTP-only cookie storage for refresh tokens.
  * **What to try next:** Add strict quality criteria for modern security practices.

---

### Version 4: Adding Quality Criteria
* **Prompt:** `Write backend code for user authentication. The project is an existing FastAPI (Python) application using PostgreSQL and SQLAlchemy. The goal is to implement production-grade JWT authentication with login, registration, and access/refresh token rotation. Format the output as modular files matching a standard FastAPI project structure (e.g., schemas.py, models.py, router.py, security.py). Quality criteria: Use bcrypt directly (or Argon2 via pwdlib), store refresh tokens in HTTP-only cookies, and explicitly handle database exceptions.`
* **Output Excerpt:**
  > "### `app/security.py`
  > ```python
  > import bcrypt
  > def hash_password(password: str) -> str:
  >     return bcrypt.hashpw(password.encode('utf-8'), bcrypt.gensalt()).decode('utf-8')
  > ```
  > ### `app/routers/auth.py`
  > ```python
  > response.set_cookie(key="refresh_token", value=refresh_token, httponly=True, samesite="lax")
  > ```"
* **Notes:**
  * **What changed in the prompt:** Added Quality Criteria (direct `bcrypt`/`pwdlib` usage, HTTP-only cookies, DB exception handling).
  * **What improved in the output:** The output stopped relying on legacy libraries, correctly protected refresh tokens against XSS by using HTTP-only cookies, and wrapped DB sessions in try-except blocks.
  * **What still failed:** It included unnecessary basic explanations of what JWTs are and step-by-step instructions on how to install Python.
  * **What to try next:** Add constraints to eliminate conversational fluff and target an experienced developer audience.

---

### Version 5: Adding Constraints
* **Prompt:** `Write backend code for user authentication. The project is an existing FastAPI (Python) application using PostgreSQL and SQLAlchemy. The goal is to implement production-grade JWT authentication with login, registration, and access/refresh token rotation. Format the output as modular files matching a standard FastAPI project structure (e.g., schemas.py, models.py, router.py, security.py). Quality criteria: Use bcrypt directly (or Argon2 via pwdlib), store refresh tokens in HTTP-only cookies, and explicitly handle database exceptions. Constraints: Omit high-level conceptual explanations, pip install steps, or basic setup tutorials. Provide production code and inline code comments only.`
* **Output Excerpt:**
  > "### `app/security.py`
  > ```python
  > # Hashing and JWT verification utilities
  > ...
  > ```
  > ### `app/models.py`
  > ```python
  > # User and Token database models
  > ...
  > ```"
* **Notes:**
  * **What changed in the prompt:** Added Constraints (no setup tutorials, no conceptual explanations, code-only output).
  * **What improved in the output:** The response stripped away 300+ words of introductory text, conceptual hand-waving, and installation commands, giving a 100% clean, professional developer resource.
  * **What still failed:** Nothing critical for this scope; the output is directly usable by an engineer.
  * **What to try next:** Final cleanup so any developer on the team can reuse it seamlessly.

---

### Final Reusable Prompt (Prompt Ladder)
```text
Act as a Senior Database Engineer specializing in high-performance PostgreSQL and multi-tenant architectures.

Task: Design a production-grade relational database schema and Prisma migration for [INSERT DOMAIN HERE, e.g., Multi-Tenant B2B Authentication].

Context & Constraints:
- Architecture: [e.g., Multi-tenant SaaS with row-level tenant isolation]
- Key Features: [e.g., Dynamic RBAC, Refresh Token Rotation with replay detection, Audit Logging]
- Compliance/Quality: Foreign keys must include cascading rules, indexes must be explicit for all FK lookups, timestamp fields must use TIMESTAMPTZ.

Output Format:
1. PostgreSQL DDL SQL block wrapped inside a `BEGIN; ... COMMIT;` transaction block.
2. Equivalent Prisma Schema (`schema.prisma`).
3. Indexing Strategy Summary Table (Columns: Table | Index Name | Target Columns | Rationale).

Step-by-Step Instructions:
1. Model core identity and organizational entity tables.
2. Model authorization structures (roles, permissions, join tables).
3. Model session management with explicit security safeguards (token hashing, expiry, rotation tracking).
4. Generate the PostgreSQL DDL script with explicit constraints and indexes.
5. Translate the models into a valid `schema.prisma` file.

<FollowUp label="Want to convert this into a ready-to-push GitHub workflow or PR template?" query="Yes, format this document as a GitHub PR description template for submitting FL-01 deliverables."/>
