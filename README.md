# ServeRest API Test Suite — QA 5X Challenge

[![Badge ServeRest](https://img.shields.io/badge/API-ServeRest-green)](https://github.com/ServeRest/ServeRest/)
[![Postman](https://img.shields.io/badge/Tested%20with-Postman-FF6C37?logo=postman&logoColor=white)](https://www.postman.com/)

Automated API test collection for the [ServeRest](https://serverest.dev) virtual store API, built as part of the **QA 5X Technical Challenge**.

---

## 📁 Files

| File | Description |
|---|---|
| `ServeRest_API_Tests.postman_collection.json` | Full test collection (22 test cases) |
| `ServeRest_ENV.postman_environment.json` | Environment with all required variables |
| `README.md` | This file |

---

## 🧪 Test Coverage

| Suite | Test Cases | Scenarios |
|---|---|---|
| 🔐 Auth | TC-AUTH-01 to 03 | Valid login, wrong password, missing field |
| 👤 Users | TC-USER-01 to 07 | List, CRUD full cycle, duplicate email, 404 after delete |
| 📦 Products | TC-PROD-01 to 07 | List, CRUD full cycle, duplicate name, non-admin blocked |
| 🛒 Cart | TC-CART-00 to 08 | List, create, duplicate rejected, checkout, cancel, teardown |

**Total: 22 test cases, 60+ assertions**

---

## 🚀 How to Run

### Prerequisites

- [Postman](https://www.postman.com/downloads/) desktop app (v10+) **or** [Newman](https://github.com/postmanlabs/newman) CLI
- Internet connection (tests run against `https://serverest.dev`)

> ⚠️ No additional dependencies, no `.env` files, no code to compile.  
> All environment variables are **set and cleaned up automatically** by the collection scripts.

---

### Option 1 — Postman Desktop (GUI)

1. Open **Postman**.
2. Click **Import** → drag and drop both files:
   - `ServeRest_API_Tests.postman_collection.json`
   - `ServeRest_ENV.postman_environment.json`
3. In the top-right dropdown, select **ServeRest ENV**.
4. In the sidebar, right-click the collection **ServeRest API Tests — QA 5X Challenge** → **Run collection**.
5. In the Collection Runner:
   - Keep all requests selected.
   - Set **Delay** to `200ms` (recommended to avoid race conditions).
   - Click **Run ServeRest API Tests — QA 5X Challenge**.

---

### Option 2 — Newman CLI

```bash
# Install Newman globally
npm install -g newman

# Run the collection
newman run ServeRest_API_Tests.postman_collection.json \
  -e ServeRest_ENV.postman_environment.json \
  --delay-request 200 \
  --reporters cli,json \
  --reporter-json-export results.json
```

#### With HTML report (optional)

```bash
npm install -g newman-reporter-htmlextra

newman run ServeRest_API_Tests.postman_collection.json \
  -e ServeRest_ENV.postman_environment.json \
  --delay-request 200 \
  --reporters cli,htmlextra \
  --reporter-htmlextra-export report.html
```

Then open `report.html` in your browser.

---

## 🏗️ Architecture & Design Decisions

### Self-contained & stateless
Every run starts from zero. The pre-request scripts create all required data (users, products, carts) and the test scripts clean up after themselves. No fixture files, no seed data, no shared state between runs.

### Execution order matters
The collection is designed to run **top-to-bottom** in sequence. Later suites depend on tokens and IDs set by earlier ones:

```
TC-AUTH-01 (creates admin + stores token)
  └─> TC-PROD-02 (uses bearerToken to create product)
       └─> TC-CART-02 (uses cartProductId to create cart)
            └─> TC-CART-05 (checkout)
                 └─> TC-CART-07 (cancel after recreate)
                      └─> TC-CART-08 (teardown / cleanup)
```

### Dynamic data via timestamps
All emails and product names include `Date.now()` to guarantee uniqueness per run, avoiding conflicts from shared environment data.

### Design patterns used
- **Setup / Teardown steps** — dedicated setup requests (TC-CART-00, TC-CART-06) and a teardown (TC-CART-08) isolate state management from business assertions.
- **Chained requests via `pm.sendRequest`** — complex pre-conditions (e.g., creating a user then logging in as that user) are handled inline without breaking the sequential runner flow.
- **Environment variables as test context** — IDs and tokens flow through `pm.environment.set/get` rather than hardcoded values.

---

## ✅ Assertion Strategy

Each test case has multiple focused assertions:

| Assertion type | Example |
|---|---|
| Status code | `pm.response.to.have.status(201)` |
| Response message | `pm.expect(body.message).to.equal('Cadastro realizado com sucesso')` |
| Schema validation | `pm.expect(body).to.have.all.keys('nome', 'email', 'password', 'administrador', '_id')` |
| Data integrity | `pm.expect(body._id).to.equal(pm.environment.get('newUserId'))` |
| Business rule | `pm.expect(body.message).to.equal('Não é permitido ter mais de 1 carrinho')` |
| Performance | `pm.expect(pm.response.responseTime).to.be.below(2000)` |
| Negative (absence) | `pm.expect(body).to.not.have.property('authorization')` |

---

## 🌐 Base URL

```
https://serverest.dev
```

To run against a local ServeRest instance:

```bash
npx serverest
```

Then change the `baseUrl` in `ServeRest ENV` to `http://localhost:3000`.

---

## 📋 API Reference

Full Swagger documentation: [https://serverest.dev](https://serverest.dev)

| Resource | Endpoint |
|---|---|
| Login | `POST /login` |
| Users | `GET/POST /usuarios`, `GET/PUT/DELETE /usuarios/{id}` |
| Products | `GET/POST /produtos`, `GET/PUT/DELETE /produtos/{id}` |
| Carts | `GET/POST /carrinhos`, `GET /carrinhos/{id}`, `DELETE /carrinhos/concluir-compra`, `DELETE /carrinhos/cancelar-compra` |

---

*Built for the QA 5X Technical Challenge.*
