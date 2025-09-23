# Complete Ingress.yaml Breakdown with Detailed Examples

I'll provide a comprehensive explanation of your ingress configuration, showing exactly how traffic flows and how URLs are processed.

## 🏗️ **Overall Architecture**

Your ingress file creates **TWO separate ingress resources** that work together to route traffic:

1. **`todo-api-ingress`** - Handles API traffic with URL rewriting
2. **`todo-frontend-ingress`** - Handles frontend traffic without rewriting

---

## 📋 **Ingress Resource #1: API Ingress**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: todo-api-ingress
  annotations:
    nginx.ingress.kubernetes.io/use-regex: "true"
    nginx.ingress.kubernetes.io/rewrite-target: /$2
spec:
  rules:
    - http:
        paths:
          - path: /api(/|$)(.*)
            pathType: ImplementationSpecific
            backend:
              service:
                name: backend-service
                port:
                  number: 8000
```

### **🔍 Component Breakdown**

#### **Annotations Explained**

```yaml
annotations:
  nginx.ingress.kubernetes.io/use-regex: "true"
  nginx.ingress.kubernetes.io/rewrite-target: /$2
```

- **`use-regex: "true"`**: Enables regex pattern matching in the path
- **`rewrite-target: /$2`**: Rewrites the URL using the second capture group from the regex

#### **Path Pattern Analysis**

```yaml
path: /api(/|$)(.*)
```

**Regex Breakdown**:

- `/api` - Literal match for "/api"
- `(/|$)` - **First capture group ($1)**: Matches either "/" or end of string
- `(.*)` - **Second capture group ($2)**: Matches everything that follows

### **🌐 Traffic Flow Examples**

#### **Example 1: GET /api**

```
1. Browser Request: GET /api
2. Pattern Match: /api(/|$)(.*)
   - Literal: /api ✓
   - Group 1 ($1): (empty - matches end of string)
   - Group 2 ($2): (empty)
3. Rewrite Rule: /$2 = / (empty becomes root)
4. Backend Receives: GET /
5. Go Backend Handler: app.Get("/", ...) responds with API status
```

#### **Example 2: GET /api/**

```
1. Browser Request: GET /api/
2. Pattern Match: /api(/|$)(.*)
   - Literal: /api ✓
   - Group 1 ($1): /
   - Group 2 ($2): (empty)
3. Rewrite Rule: /$2 = / (empty becomes root)
4. Backend Receives: GET /
5. Go Backend Handler: app.Get("/", ...) responds with API status
```

#### **Example 3: GET /api/todos**

```
1. Browser Request: GET /api/todos
2. Pattern Match: /api(/|$)(.*)
   - Literal: /api ✓
   - Group 1 ($1): /
   - Group 2 ($2): todos
3. Rewrite Rule: /$2 = /todos
4. Backend Receives: GET /todos
5. Go Backend Handler: app.Get("/todos", ...) returns todo list
```

#### **Example 4: POST /api/todos**

```
1. Browser Request: POST /api/todos
2. Pattern Match: /api(/|$)(.*)
   - Literal: /api ✓
   - Group 1 ($1): /
   - Group 2 ($2): todos
3. Rewrite Rule: /$2 = /todos
4. Backend Receives: POST /todos
5. Go Backend Handler: app.Post("/todos", ...) creates new todo
```

#### **Example 5: PATCH /api/todos/update/123**

```
1. Browser Request: PATCH /api/todos/update/123
2. Pattern Match: /api(/|$)(.*)
   - Literal: /api ✓
   - Group 1 ($1): /
   - Group 2 ($2): todos/update/123
3. Rewrite Rule: /$2 = /todos/update/123
4. Backend Receives: PATCH /todos/update/123
5. Go Backend Handler: app.Patch("/todos/update/:id", ...) toggles todo completion
```

### **🎯 What the Backend Sees**

| Original Frontend Call        | What Backend Receives     | Backend Handler                       |
| ----------------------------- | ------------------------- | ------------------------------------- |
| `GET /api`                    | `GET /`                   | `app.Get("/", ...)`                   |
| `GET /api/`                   | `GET /`                   | `app.Get("/", ...)`                   |
| `GET /api/todos`              | `GET /todos`              | `app.Get("/todos", ...)`              |
| `POST /api/todos`             | `POST /todos`             | `app.Post("/todos", ...)`             |
| `PATCH /api/todos/update/123` | `PATCH /todos/update/123` | `app.Patch("/todos/update/:id", ...)` |
| `DELETE /api/todos/456`       | `DELETE /todos/456`       | `app.Delete("/todos/:id", ...)`       |

---

## 📋 **Ingress Resource #2: Frontend Ingress**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: todo-frontend-ingress
  # NO annotations - no rewriting!
spec:
  rules:
    - http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend-service
                port:
                  number: 3000
```

### **🔍 Component Breakdown**

#### **No Annotations**

- **No rewrite rules**: URLs are passed through unchanged
- **No regex matching**: Simple prefix matching only

#### **Path Pattern**

```yaml
path: /
pathType: Prefix
```

- **`path: /`**: Matches ANY path that starts with "/"
- **`pathType: Prefix`**: Matches all paths that begin with the specified prefix
- **Catch-all behavior**: Any request not matched by the API ingress comes here

### **🌐 Traffic Flow Examples**

#### **Example 1: Frontend Homepage**

```
1. Browser Request: GET /
2. Path Match: / (prefix match) ✓
3. No Rewrite: URL unchanged
4. Frontend Receives: GET /
5. Next.js serves: Homepage (page.tsx)
```

#### **Example 2: JavaScript Chunks**

```
1. Browser Request: GET /_next/static/chunks/app/page-abc123.js
2. Path Match: / (prefix match) ✓
3. No Rewrite: URL unchanged
4. Frontend Receives: GET /_next/static/chunks/app/page-abc123.js
5. Next.js serves: JavaScript file with correct Content-Type
```

#### **Example 3: CSS Files**

```
1. Browser Request: GET /_next/static/css/app-layout.css
2. Path Match: / (prefix match) ✓
3. No Rewrite: URL unchanged
4. Frontend Receives: GET /_next/static/css/app-layout.css
5. Next.js serves: CSS file
```

#### **Example 4: Any Frontend Route**

```
1. Browser Request: GET /about
2. Path Match: / (prefix match) ✓
3. No Rewrite: URL unchanged
4. Frontend Receives: GET /about
5. Next.js handles: Client-side routing (React Router)
```

### **🎯 What the Frontend Sees**

| Browser Request                            | What Frontend Receives                     | Next.js Behavior    |
| ------------------------------------------ | ------------------------------------------ | ------------------- |
| `GET /`                                    | `GET /`                                    | Serves homepage     |
| `GET /_next/static/chunks/app/page-xyz.js` | `GET /_next/static/chunks/app/page-xyz.js` | Serves JS file      |
| `GET /_next/static/css/styles.css`         | `GET /_next/static/css/styles.css`         | Serves CSS file     |
| `GET /about`                               | `GET /about`                               | Client-side routing |
| `GET /favicon.ico`                         | `GET /favicon.ico`                         | Serves favicon      |

---

## 🚦 **Request Routing Priority and Logic**

### **How NGINX Decides Which Ingress to Use**

NGINX processes ingress rules based on **specificity** and **order**:

1. **Most Specific Patterns First**: `/api(/|$)(.*)` is more specific than `/`
2. **Regex Patterns**: Have higher priority than simple prefix matches
3. **Longest Match Wins**: More specific paths take precedence

### **Decision Tree**

```
Incoming Request
       ↓
Does it match /api(/|$)(.*)?
       ↓                    ↓
     YES                   NO
       ↓                    ↓
todo-api-ingress      todo-frontend-ingress
       ↓                    ↓
Apply rewrite /$2      No rewrite
       ↓                    ↓
backend-service:8000   frontend-service:3000
```

### **Routing Examples with Decision Process**

#### **API Request Flow**

```
1. Request: GET /api/todos
2. Check: Does "/api/todos" match "/api(/|$)(.*)"?
   → YES (matches /api + / + todos)
3. Route to: todo-api-ingress
4. Apply rewrite: /$2 = /todos
5. Send to: backend-service:8000
6. Backend receives: GET /todos
```

#### **Frontend Request Flow**

```
1. Request: GET /_next/static/chunks/main.js
2. Check: Does "/_next/static/chunks/main.js" match "/api(/|$)(.*)"?
   → NO (doesn't start with /api)
3. Check: Does "/_next/static/chunks/main.js" match "/"?
   → YES (starts with /)
4. Route to: todo-frontend-ingress
5. No rewrite applied
6. Send to: frontend-service:3000
7. Frontend receives: GET /_next/static/chunks/main.js
```

---

## 🔄 **Complete Request/Response Cycle Examples**

### **Example 1: Creating a Todo**

```
🌐 Frontend JavaScript:
fetch("/api/todos", {
  method: "POST",
  body: JSON.stringify({body: "Learn Kubernetes", completed: false})
})

⬇️ NGINX Ingress Processing:
1. Receives: POST /api/todos
2. Matches: /api(/|$)(.*) → Groups: $1="/", $2="todos"
3. Rewrites: /$2 → /todos
4. Routes to: backend-service:8000

⬇️ Go Backend:
1. Receives: POST /todos
2. Handler: app.Post("/todos", func(c *fiber.Ctx) error {...})
3. Inserts into PostgreSQL
4. Returns: {"id": 1, "body": "Learn Kubernetes", "completed": false}

⬆️ Response Path:
1. Backend → NGINX → Frontend
2. Frontend updates state with new todo
```

### **Example 2: Loading Frontend Assets**

```
🌐 Browser:
<script src="/_next/static/chunks/app/page-abc123.js"></script>

⬇️ NGINX Ingress Processing:
1. Receives: GET /_next/static/chunks/app/page-abc123.js
2. No match: /api(/|$)(.*)
3. Matches: / (prefix)
4. No rewrite applied
5. Routes to: frontend-service:3000

⬇️ Next.js Frontend:
1. Receives: GET /_next/static/chunks/app/page-abc123.js
2. Serves: Pre-built JavaScript file
3. Content-Type: application/javascript

⬆️ Response:
1. Browser receives JavaScript code
2. Executes without syntax errors
```

---

## ⚙️ **Configuration Benefits**

### **Why Split Ingress Resources?**

#### **❌ Single Ingress Problems (What We Fixed)**

```yaml
# This was problematic:
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2 # Applied to ALL paths!
spec:
  rules:
    - http:
        paths:
          - path: /api(/|$)(.*) # Needs rewrite ✓
          - path: / # Should NOT be rewritten ✗
```

**Problem**: Static assets like `/_next/static/chunks/app/page-xyz.js` were being rewritten, returning HTML instead of JavaScript.

#### **✅ Split Ingress Benefits**

1. **Selective Rewriting**: Only API paths get rewritten
2. **Static Asset Preservation**: Frontend assets served unchanged
3. **Clear Separation**: API and frontend concerns separated
4. **Debugging Clarity**: Each ingress has single responsibility

### **Performance Implications**

1. **NGINX Efficiency**: Two ingress resources don't significantly impact performance
2. **Caching**: Static assets can be cached properly with correct headers
3. **Load Balancing**: Each service can be scaled independently

---

## 🛠️ **Advanced Configuration Options**

### **Additional Annotations You Could Add**

#### **For API Ingress**:

```yaml
annotations:
  nginx.ingress.kubernetes.io/use-regex: "true"
  nginx.ingress.kubernetes.io/rewrite-target: /$2
  nginx.ingress.kubernetes.io/cors-allow-origin: "*"
  nginx.ingress.kubernetes.io/cors-allow-methods: "GET, POST, PATCH, DELETE"
  nginx.ingress.kubernetes.io/cors-allow-headers: "Content-Type"
```

#### **For Frontend Ingress**:

```yaml
annotations:
  nginx.ingress.kubernetes.io/cache-control: "public, max-age=31536000" # Cache static assets
  nginx.ingress.kubernetes.io/expires: "1y"
```

### **Alternative Path Strategies**

#### **More Specific API Paths**:

```yaml
# Instead of catch-all, could be more specific:
- path: /api/todos(/.*)?
- path: /api/auth(/.*)?
- path: /api/users(/.*)?
```

---

## 🔍 **Troubleshooting Guide**

### **Common Issues and Diagnostics**

#### **1. Check Which Ingress Matches**

```bash
# Test specific paths
curl -I http://localhost:8080/api/todos
curl -I http://localhost:8080/_next/static/chunks/main.js

# Check ingress status
kubectl describe ingress todo-api-ingress
kubectl describe ingress todo-frontend-ingress
```

#### **2. Verify Rewrite Rules**

```bash
# Check backend receives correct path
kubectl logs -l app=backend | grep "GET\|POST\|PATCH\|DELETE"
```

#### **3. Test Path Matching**

```bash
# Should go to API ingress
curl http://localhost:8080/api
curl http://localhost:8080/api/
curl http://localhost:8080/api/todos

# Should go to frontend ingress
curl http://localhost:8080/
curl http://localhost:8080/_next/static/chunks/main.js
curl http://localhost:8080/about
```

---

## 🎯 **Key Takeaways**

### **1. Path Matching Priority**

- Regex patterns (`/api(/|$)(.*)`) are more specific than prefix patterns (`/`)
- Most specific match wins

### **2. Rewrite Rules**

- Only apply rewrite rules where needed (API paths)
- Static assets must be served without modification

### **3. Service Separation**

- API ingress handles `/api/*` → backend-service:8000
- Frontend ingress handles everything else → frontend-service:3000

### **4. URL Processing**

- API calls: `/api/todos` → Backend sees `/todos`
- Static assets: `/_next/static/file.js` → Frontend sees `/_next/static/file.js`
- Frontend routes: `/about` → Frontend handles client-side routing

This split ingress architecture provides **clean separation of concerns**, **proper URL handling**, and **correct content serving** for both your API and frontend static assets.
