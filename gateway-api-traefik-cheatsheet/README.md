# Gateway API & Traefik Cheat Sheet

> **Quick Reference Guide for Kubernetes Gateway API with Traefik Controller**
>
> **Example Application:** Online Bookstore (`bookstore.example.com`)
>
> **Legend:**
>
> - ✅ **Native Gateway API** - Portable across all Gateway API controllers
> - 🔧 **Traefik-Specific** - Only works with Traefik controller (vendor extension)

---

## Table of Contents

1. [Application Overview](#application-overview)
2. [Path Matching](#path-matching)
3. [Native Gateway API Filters](#native-gateway-api-filters)
4. [Traefik Middleware](#traefik-middleware)
5. [Complete HTTPRoute Examples](#complete-httproute-examples)
6. [Common Patterns](#common-patterns)
7. [Flask Application Example](#flask-application-example)
8. [Troubleshooting](#troubleshooting)

---

## Application Overview

**Bookstore Application Architecture:**

```
bookstore.example.com
├── /                          → Homepage (frontend)
├── /books                     → Browse books (frontend)
├── /books/<id>                → Book details (frontend)
├── /cart                      → Shopping cart (frontend)
├── /checkout                  → Checkout page (frontend)
├── /search                    → Search page (frontend)
├── /api/v1/*                  → API v1 (backend)
├── /api/v2/*                  → API v2 (backend - newer)
├── /admin/*                   → Admin dashboard (protected)
├── /static/*                  → Static assets (CDN/S3)
├── /health                    → Health check endpoint
└── /metrics                   → Prometheus metrics

admin.bookstore.example.com
└── /                          → Admin panel (separate subdomain)
```

**Kubernetes Services:**

- `bookstore-frontend` (port 3000) - React/Vue SPA serving the customer-facing site
- `bookstore-api-v1` (port 8080) - API version 1 (legacy)
- `bookstore-api-v2` (port 8080) - API version 2 (current)
- `bookstore-admin` (port 3001) - Admin dashboard
- `bookstore-cdn` (ExternalName to S3 bucket for static assets)

---

## Path Matching

### 1. PathPrefix ✅

**Matches requests where the path starts with the specified value**

```yaml
matches:
  - path:
      type: PathPrefix
      value: /api/
```

**Use Case:** Route all bookstore API requests to the backend service.

**Matches:**

- ✅ `/api/`
- ✅ `/api/books`
- ✅ `/api/v2/books/123`
- ✅ `/api/search?q=kubernetes`
- ❌ `/api` (no trailing slash - use Exact for this)

**Example:** `/api/books`, `/api/orders`, `/api/cart` all go to `bookstore-api-v2`.

---

### 2. Exact ✅

**Matches requests where the path exactly equals the specified value**

```yaml
matches:
  - path:
      type: Exact
      value: /health
```

**Use Case:** Kubernetes liveness probe endpoint - must be exactly `/health`.

**Matches:**

- ✅ `/health`
- ❌ `/health/`
- ❌ `/health/detailed`

**Example:** Health check for `bookstore-api-v2` service.

---

### 3. RegularExpression ✅ (Extended)

**Matches requests using regex patterns**

```yaml
matches:
  - path:
      type: RegularExpression
      value: ^/api/v[0-9]+/.*
```

**Use Case:** Route all versioned API endpoints to version-aware backend.

**Matches:**

- ✅ `/api/v1/books`
- ✅ `/api/v2/books`
- ✅ `/api/v99/books`
- ❌ `/api/books` (no version)

**Example:** Handle both v1 and v2 API requests with one rule.

**Note:** Not all Gateway API implementations support this (Extended feature).

---

## Native Gateway API Filters

### 1. RequestRedirect ✅

**Redirects requests to a different URL**

#### HTTP to HTTPS Redirect

```yaml
filters:
  - type: RequestRedirect
    requestRedirect:
      scheme: https
      statusCode: 301
```

**Use Case:** Force all bookstore traffic to HTTPS for secure checkout and payment processing.

---

#### Path Redirect - Trailing Slash for SPA

```yaml
matches:
  - path:
      type: Exact
      value: /books
filters:
  - type: RequestRedirect
    requestRedirect:
      path:
        type: ReplaceFullPath
        replaceFullPath: /books/
      statusCode: 308
```

**Use Case:** React app at `/books/` needs trailing slash so relative imports work (`./BookList.js`).

**Status Code Guide:**

- `301` - Permanent redirect, may change POST to GET
- `302` - Temporary redirect (maintenance mode)
- `307` - Temporary redirect, preserves HTTP method  
- `308` - Permanent redirect, preserves HTTP method ⭐ (best for SPAs/APIs)

---

### 2. URLRewrite ✅

**Rewrites the URL path before forwarding to backend**

#### ReplaceFullPath

```yaml
matches:
  - path:
      type: Exact
      value: /
filters:
  - type: URLRewrite
    urlRewrite:
      path:
        type: ReplaceFullPath
        replaceFullPath: /index.html
```

**Use Case:** Redirect root path `/` to `/index.html` for the bookstore homepage.

**Example:**

- Client requests: `/`
- Backend receives: `/index.html`

---

#### ReplacePrefixMatch

```yaml
matches:
  - path:
      type: PathPrefix
      value: /static/
filters:
  - type: URLRewrite
    urlRewrite:
      path:
        type: ReplacePrefixMatch
        replacePrefixMatch: /bookstore-assets/v1/
```

**Use Case:** External path is `/static/logo.png` but S3 bucket structure is `/bookstore-assets/v1/logo.png`.

**Example:**

- Client requests: `/static/images/logo.png`
- S3 receives: `/bookstore-assets/v1/images/logo.png`

---

### 3. RequestHeaderModifier ✅

**Adds, sets, or removes HTTP request headers**

```yaml
filters:
  - type: RequestHeaderModifier
    requestHeaderModifier:
      set:
        - name: X-Store-ID
          value: "bookstore-main"
        - name: X-Forwarded-Host
          value: "bookstore.example.com"
      add:
        - name: X-Request-ID
          value: "auto-generated-uuid"
      remove:
        - Cookie
```

**Use Case:** Add store identifier for multi-store backend. Backend uses `X-Store-ID` to route to correct database.

**Difference between `set` and `add`:**

- `set` - Replaces header if exists, adds if doesn't
- `add` - Appends to existing header (multiple values allowed)

---

### 4. ResponseHeaderModifier ✅ (Extended)

**Adds, sets, or removes HTTP response headers**

```yaml
filters:
  - type: ResponseHeaderModifier
    responseHeaderModifier:
      set:
        - name: X-Frame-Options
          value: SAMEORIGIN
        - name: X-Content-Type-Options
          value: nosniff
        - name: Strict-Transport-Security
          value: "max-age=31536000; includeSubdomains"
        - name: Content-Security-Policy
          value: "default-src 'self'; script-src 'self' 'unsafe-inline'"
      remove:
        - Server
        - X-Powered-By
```

**Use Case:** Security headers for bookstore checkout page. Remove server fingerprinting headers.

---

### 5. RequestMirror ✅ (Extended)

**Mirrors (duplicates) requests to additional backend**

```yaml
filters:
  - type: RequestMirror
    requestMirror:
      backendRef:
        name: bookstore-analytics
        port: 9000
backendRefs:
  - name: bookstore-api-v2
    port: 8080
```

**Use Case:** Send production traffic to both API v2 and analytics service for real-time sales monitoring.

---

## Traefik Middleware

### 1. Chain Middleware 🔧

**Combines multiple middlewares in sequence**

```yaml
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: bookstore-security-chain
spec:
  chain:
    middlewares:
      - name: bookstore-rate-limit
      - name: bookstore-security-headers
      - name: bookstore-error-pages
```

**Use Case:** Apply security stack (rate limiting + headers + error pages) across all bookstore routes.

**Usage in HTTPRoute:**

```yaml
filters:
  - type: ExtensionRef
    extensionRef:
      group: traefik.io
      kind: Middleware
      name: bookstore-security-chain
```

---

### 2. RedirectRegex 🔧

**Redirect based on regex pattern**

```yaml
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: www-redirect
spec:
  redirectRegex:
    regex: "^https?://bookstore.example.com/(.*)$"
    replacement: "https://www.bookstore.example.com/${1}"
    permanent: true
```

**Use Case:** Redirect `bookstore.example.com` to `www.bookstore.example.com` while preserving path and query params.

---

### 3. StripPrefix 🔧

**Remove path prefix before forwarding to backend**

```yaml
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: api-strip-prefix
spec:
  stripPrefix:
    prefixes:
      - /api/v2
      - /api
```

**Use Case:** External path is `/api/v2/books` but backend expects just `/books`.

**Example:**

- Client requests: `/api/v2/books`
- Backend receives: `/books`

---

### 4. Headers (Security) 🔧

**Set comprehensive security headers**

```yaml
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: bookstore-security-headers
spec:
  headers:
    browserXssFilter: true
    contentTypeNosniff: true
    frameDeny: false
    customFrameOptionsValue: "SAMEORIGIN"
    stsSeconds: 63072000
    stsIncludeSubdomains: true
    stsPreload: true
    contentSecurityPolicy: "default-src 'self'; img-src 'self' https://cdn.bookstore.com"
    referrerPolicy: "strict-origin-when-cross-origin"
```

**Use Case:** Protect bookstore from XSS, clickjacking, and MIME-type attacks.

---

### 5. Errors (Custom Error Pages) 🔧

**Serve custom error pages**

```yaml
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: bookstore-errors
spec:
  errors:
    status:
      - "404"
      - "503"
    query: "/{status}"
    service:
      name: bookstore-error-pages
      port: 80
```

**Use Case:** Show friendly "Book not found" page instead of ugly 404 errors.

---

### 6. RateLimit 🔧

**Limit request rate per IP**

```yaml
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: bookstore-api-rate-limit
spec:
  rateLimit:
    average: 100
    period: 1s
    burst: 50
```

**Use Case:** Protect bookstore API from abuse - 100 requests/second per IP, burst of 50.

---

### 7. BasicAuth 🔧

**HTTP Basic Authentication**

```yaml
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: bookstore-admin-auth
spec:
  basicAuth:
    secret: admin-credentials
```

**Secret (htpasswd format):**

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: admin-credentials
data:
  users: |
    admin:$apr1$H6uskkkW$IgXLP6ewTrSuBkTrqE8wj/
```

**Use Case:** Protect `/admin/*` routes with username/password.

**Generate password:** `htpasswd -nb admin secretpassword`

---

### 8. ForwardAuth 🔧

**External authentication service (OAuth/OIDC)**

```yaml
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: bookstore-oauth
spec:
  forwardAuth:
    address: "http://oauth-proxy.auth.svc.cluster.local/verify"
    trustForwardHeader: true
    authResponseHeaders:
      - X-Auth-User
      - X-Auth-Email
```

**Use Case:** OAuth2 login for bookstore customer accounts before accessing `/checkout`.

---

### 9. Compress 🔧

**Enable gzip compression**

```yaml
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: bookstore-compression
spec:
  compress:
    minResponseBodyBytes: 1024
```

**Use Case:** Compress book descriptions and images larger than 1KB for faster page loads.

---

### 10. CircuitBreaker 🔧

**Prevent cascading failures**

```yaml
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: bookstore-circuit-breaker
spec:
  circuitBreaker:
    expression: "NetworkErrorRatio() > 0.30"
```

**Use Case:** If bookstore payment gateway fails >30%, stop sending requests to allow recovery.

---

## Complete HTTPRoute Examples

### Example 1: HTTP to HTTPS + Frontend Routing

```yaml
# Redirect HTTP to HTTPS
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: bookstore-http-redirect
spec:
  parentRefs:
    - name: main-gateway
      sectionName: http
  hostnames:
    - bookstore.example.com
  rules:
    - filters:
        - type: RequestRedirect
          requestRedirect:
            scheme: https
            statusCode: 301
---
# Serve HTTPS traffic
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: bookstore-https
spec:
  parentRefs:
    - name: main-gateway
      sectionName: https
  hostnames:
    - bookstore.example.com
  rules:
    - matches:
        - path:
            type: PathPrefix
            value: /
      filters:
        - type: ExtensionRef
          extensionRef:
            group: traefik.io
            kind: Middleware
            name: bookstore-security-chain
      backendRefs:
        - name: bookstore-frontend
          port: 3000
```

**Use Case:** Force HTTPS for all bookstore traffic, apply security headers.

---

### Example 2: API Version Routing

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: bookstore-api-routes
spec:
  parentRefs:
    - name: main-gateway
  hostnames:
    - bookstore.example.com
  rules:
    # API v1 (legacy)
    - matches:
        - path:
            type: PathPrefix
            value: /api/v1/
      filters:
        - type: ExtensionRef
          extensionRef:
            group: traefik.io
            kind: Middleware
            name: bookstore-api-rate-limit
      backendRefs:
        - name: bookstore-api-v1
          port: 8080
    
    # API v2 (current)
    - matches:
        - path:
            type: PathPrefix
            value: /api/v2/
      filters:
        - type: ExtensionRef
          extensionRef:
            group: traefik.io
            kind: Middleware
            name: bookstore-api-rate-limit
      backendRefs:
        - name: bookstore-api-v2
          port: 8080
    
    # Unversioned defaults to v2
    - matches:
        - path:
            type: PathPrefix
            value: /api/
      filters:
        - type: URLRewrite
          urlRewrite:
            path:
              type: ReplacePrefixMatch
              replacePrefixMatch: /api/v2/
      backendRefs:
        - name: bookstore-api-v2
          port: 8080
```

**Use Case:** Support multiple API versions during migration. Unversioned `/api/books` → v2.

---

### Example 3: Admin Panel with Authentication

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: bookstore-admin
spec:
  parentRefs:
    - name: main-gateway
  hostnames:
    - admin.bookstore.example.com
  rules:
    - matches:
        - path:
            type: PathPrefix
            value: /
      filters:
        - type: ExtensionRef
          extensionRef:
            group: traefik.io
            kind: Middleware
            name: bookstore-admin-auth
        - type: ExtensionRef
          extensionRef:
            group: traefik.io
            kind: Middleware
            name: bookstore-security-headers
      backendRefs:
        - name: bookstore-admin
          port: 3001
```

**Use Case:** Admin dashboard at separate subdomain, protected with BasicAuth.

---

### Example 4: Canary Deployment (10% Traffic Split)

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: bookstore-canary
spec:
  parentRefs:
    - name: main-gateway
  hostnames:
    - bookstore.example.com
  rules:
    - matches:
        - path:
            type: PathPrefix
            value: /api/v2/
      backendRefs:
        - name: bookstore-api-v2-stable
          port: 8080
          weight: 90
        - name: bookstore-api-v2-canary
          port: 8080
          weight: 10
```

**Use Case:** Test new features on 10% of traffic before full rollout.

---

### Example 5: Static Assets from S3

```yaml
# ExternalName service for S3
apiVersion: v1
kind: Service
metadata:
  name: bookstore-cdn
spec:
  type: ExternalName
  externalName: bookstore-assets.s3.amazonaws.com
  ports:
    - port: 443
---
# Middleware for S3 Host header
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: s3-headers
spec:
  headers:
    customRequestHeaders:
      Host: bookstore-assets.s3.amazonaws.com
---
# IngressRoute (Gateway API doesn't support ExternalName yet)
apiVersion: traefik.io/v1alpha1
kind: IngressRoute
metadata:
  name: bookstore-static-assets
spec:
  routes:
    - match: Host(`bookstore.example.com`) && PathPrefix(`/static/`)
      kind: Rule
      priority: 100
      middlewares:
        - name: s3-headers
      services:
        - name: bookstore-cdn
          port: 443
          passHostHeader: false
```

**Use Case:** Serve book cover images from S3 bucket through bookstore domain.

---

### Example 6: Health Check Endpoint

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: bookstore-health
spec:
  parentRefs:
    - name: main-gateway
  hostnames:
    - bookstore.example.com
  rules:
    - matches:
        - path:
            type: Exact
            value: /health
      backendRefs:
        - name: bookstore-api-v2
          port: 8080
```

**Use Case:** Kubernetes liveness/readiness probes for API service.

---

### Example 7: Search with Path Rewrite

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: bookstore-search
spec:
  parentRefs:
    - name: main-gateway
  hostnames:
    - bookstore.example.com
  rules:
    - matches:
        - path:
            type: PathPrefix
            value: /search
      filters:
        - type: URLRewrite
          urlRewrite:
            path:
              type: ReplacePrefixMatch
              replacePrefixMatch: /api/v2/search
      backendRefs:
        - name: bookstore-api-v2
          port: 8080
```

**Use Case:** User-friendly `/search?q=kafka` maps to `/api/v2/search?q=kafka` backend.

---

### Example 8: Trailing Slash Normalization

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: bookstore-books-route
spec:
  parentRefs:
    - name: main-gateway
  hostnames:
    - bookstore.example.com
  rules:
    # Redirect without trailing slash
    - matches:
        - path:
            type: Exact
            value: /books
      filters:
        - type: RequestRedirect
          requestRedirect:
            path:
              type: ReplaceFullPath
              replaceFullPath: /books/
            statusCode: 308
    
    # Serve the React app
    - matches:
        - path:
            type: PathPrefix
            value: /books/
      backendRefs:
        - name: bookstore-frontend
          port: 3000
```

**Use Case:** React SPA at `/books/` needs trailing slash for relative imports to work.

---

## Common Patterns

### Pattern 1: Complete Security Stack

```yaml
# Individual middlewares
---
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: bookstore-rate-limit
spec:
  rateLimit:
    average: 100
    period: 1s
    burst: 50
---
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: bookstore-security-headers
spec:
  headers:
    stsSeconds: 31536000
    stsIncludeSubdomains: true
    contentTypeNosniff: true
    browserXssFilter: true
    customFrameOptionsValue: "SAMEORIGIN"
---
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: bookstore-error-pages
spec:
  errors:
    status:
      - "404"
      - "500"
      - "503"
    service:
      name: bookstore-error-service
      port: 80
---
# Chain them
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: bookstore-security-chain
spec:
  chain:
    middlewares:
      - name: bookstore-rate-limit
      - name: bookstore-security-headers
      - name: bookstore-error-pages
```

**Use Case:** Apply to all production bookstore routes for comprehensive protection.

---

### Pattern 2: Blue-Green Deployment

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: bookstore-blue-green
spec:
  parentRefs:
    - name: main-gateway
  hostnames:
    - bookstore.example.com
  rules:
    - matches:
        - path:
            type: PathPrefix
            value: /api/v2/
      backendRefs:
        # Toggle between deployments
        - name: bookstore-api-blue
          port: 8080
          weight: 100  # Active
        - name: bookstore-api-green
          port: 8080
          weight: 0    # Standby
```

**Use Case:** Instant cutover from blue to green by swapping weights (100↔0).

---

### Pattern 3: Multi-Environment Routing

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: bookstore-staging
spec:
  parentRefs:
    - name: main-gateway
  hostnames:
    - staging.bookstore.example.com
  rules:
    - matches:
        - path:
            type: PathPrefix
            value: /
      filters:
        - type: RequestHeaderModifier
          requestHeaderModifier:
            set:
              - name: X-Environment
                value: "staging"
      backendRefs:
        - name: bookstore-frontend-staging
          port: 3000
```

**Use Case:** Staging environment with separate subdomain pointing to staging services.

---

## Flask Application Example

Here's a Flask app that implements the bookstore API routes from this cheat sheet:

**bookstore_api.py:**

```python
from flask import Flask, jsonify, request

app = Flask(__name__)

# In-memory data store (use database in production)
BOOKS = [
    {"id": 1, "title": "Kubernetes in Action", "author": "Marko Lukša", "price": 49.99, "inStock": True},
    {"id": 2, "title": "Cloud Native DevOps with Kubernetes", "author": "John Arundel", "price": 39.99, "inStock": True},
    {"id": 3, "title": "The DevOps Handbook", "author": "Gene Kim", "price": 44.99, "inStock": False},
]

# Health check endpoint
@app.route('/health')
def health():
    return jsonify({"status": "healthy", "service": "bookstore-api-v2"}), 200

# Metrics endpoint (Prometheus format)
@app.route('/metrics')
def metrics():
    return """# HELP bookstore_requests_total Total requests
# TYPE bookstore_requests_total counter
bookstore_requests_total 1234
# HELP bookstore_books_total Total books in catalog
# TYPE bookstore_books_total gauge
bookstore_books_total 3
""", 200, {'Content-Type': 'text/plain'}

# API v2 - List all books
@app.route('/api/v2/books', methods=['GET'])
def get_books():
    return jsonify({"books": BOOKS, "total": len(BOOKS)})

# API v2 - Get specific book
@app.route('/api/v2/books/<int:book_id>', methods=['GET'])
def get_book(book_id):
    book = next((b for b in BOOKS if b['id'] == book_id), None)
    if book:
        return jsonify(book)
    return jsonify({"error": "Book not found"}), 404

# API v2 - Search books
@app.route('/api/v2/search', methods=['GET'])
def search_books():
    query = request.args.get('q', '').lower()
    results = [
        {**book, "relevance": 0.95 if query in book['title'].lower() else 0.5}
        for book in BOOKS 
        if query in book['title'].lower() or query in book['author'].lower()
    ]
    return jsonify({"query": query, "results": results, "count": len(results)})

# API v2 - Shopping cart (simplified)
@app.route('/api/v2/cart', methods=['GET', 'POST'])
def cart():
    if request.method == 'GET':
        # Return empty cart for demo
        return jsonify({"items": [], "total": 0.0})
    else:
        # Add item to cart
        data = request.get_json()
        return jsonify({
            "message": "Item added to cart",
            "bookId": data.get('bookId'),
            "quantity": data.get('quantity', 1)
        }), 201

# API v2 - Orders
@app.route('/api/v2/orders', methods=['GET', 'POST'])
def orders():
    if request.method == 'GET':
        # Return empty order list for demo
        return jsonify({"orders": []})
    else:
        # Create new order
        data = request.get_json()
        return jsonify({
            "orderId": 12345,
            "status": "confirmed",
            "total": data.get('total', 0.0),
            "items": data.get('items', [])
        }), 201

# Homepage redirect (used with URLRewrite in Gateway)
@app.route('/index.html')
def index():
    return jsonify({
        "message": "Welcome to the Bookstore API",
        "version": "v2",
        "endpoints": {
            "books": "/api/v2/books",
            "search": "/api/v2/search?q=keyword",
            "cart": "/api/v2/cart",
            "orders": "/api/v2/orders"
        }
    })

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8080, debug=True)
```

**requirements.txt:**

```txt
Flask==3.0.0
gunicorn==21.2.0
```

**Dockerfile:**

```dockerfile
FROM python:3.13-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY bookstore_api.py .

EXPOSE 8080

CMD ["gunicorn", "--bind", "0.0.0.0:8080", "--workers", "4", "bookstore_api:app"]
```

**Kubernetes Deployment:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: bookstore-api-v2
  namespace: bookstore
spec:
  replicas: 3
  selector:
    matchLabels:
      app: bookstore-api-v2
  template:
    metadata:
      labels:
        app: bookstore-api-v2
        version: v2
    spec:
      containers:
      - name: api
        image: your-registry/bookstore-api:v2
        ports:
        - containerPort: 8080
        env:
        - name: FLASK_ENV
          value: "production"
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 30
        readinessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 10
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "200m"
---
apiVersion: v1
kind: Service
metadata:
  name: bookstore-api-v2
  namespace: bookstore
spec:
  selector:
    app: bookstore-api-v2
  ports:
    - port: 8080
      targetPort: 8080
      protocol: TCP
  type: ClusterIP
```

**Build and Deploy:**

```bash
# Build Docker image
docker build -t your-registry/bookstore-api:v2 .

# Push to registry
docker push your-registry/bookstore-api:v2

# Create namespace
kubectl create namespace bookstore

# Deploy
kubectl apply -f deployment.yaml

# Test locally first
python bookstore_api.py
curl http://localhost:8080/health
curl http://localhost:8080/api/v2/books
curl "http://localhost:8080/api/v2/search?q=kubernetes"
```

**Test through Gateway:**

```bash
# Health check
curl https://bookstore.example.com/health

# List books
curl https://bookstore.example.com/api/v2/books

# Get specific book
curl https://bookstore.example.com/api/v2/books/1

# Search (uses URLRewrite - /search → /api/v2/search)
curl "https://bookstore.example.com/search?q=kubernetes"

# View metrics
curl https://bookstore.example.com/metrics
```

---

## Troubleshooting

### Issue 1: Route Not Matching

**Symptoms:** Requests return 404 even though route is configured

**Debug:**

```bash
# Check HTTPRoute status
kubectl get httproute -n bookstore

# Describe for events
kubectl describe httproute bookstore-api-routes -n bookstore

# Check Gateway status
kubectl get gateway main-gateway -n gateway-system -o yaml
```

**Common Causes:**

- Missing trailing slash (`/api` vs `/api/`)
- Rule order (first match wins - check PathPrefix order)
- Namespace mismatch between HTTPRoute and Gateway
- Gateway not accepting routes from namespace

---

### Issue 2: Middleware Not Applied

**Symptoms:** Security headers missing, rate limiting not working

**Debug:**

```bash
# Verify middleware exists
kubectl get middleware -n bookstore

# Check reference is correct
kubectl get httproute bookstore-api-routes -o yaml | grep -A 10 extensionRef

# Test headers
curl -I https://bookstore.example.com/api/v2/books
```

**Common Causes:**

- Typo in middleware name
- Wrong namespace in extensionRef
- Middleware not in chain
- CRD provider not enabled in Traefik config

---

### Issue 3: 404 for All Requests

**Symptoms:** Every request returns 404

**Debug:**

```bash
# Check Gateway listeners
kubectl get gateway main-gateway -o jsonpath='{.status.listeners}'

# Verify parentRefs match
kubectl get httproute -o yaml | grep -A 5 parentRefs
```

**Common Causes:**

- `parentRefs.name` doesn't match Gateway name
- Hostname mismatch between HTTPRoute and Gateway listener
- sectionName wrong (e.g., "https" vs "http")

---

### Issue 4: Redirect Loop

**Symptoms:** Browser shows "Too many redirects"

**Common Causes:**

- Both HTTP and HTTPS HTTPRoutes redirecting
- Backend returning redirect that conflicts with Gateway redirect

**Fix:**

```yaml
# HTTP listener - redirect only
- sectionName: http
  rules:
    - filters:
        - type: RequestRedirect
          requestRedirect:
            scheme: https

# HTTPS listener - serve content (no redirect)
- sectionName: https
  rules:
    - backendRefs:
        - name: bookstore-frontend
```

---

### Issue 5: S3/External Service Returns 403

**Symptoms:** Static assets from S3 return 403 Forbidden

**Fix:** Ensure `passHostHeader: false` and correct Host header:

```yaml
spec:
  headers:
    customRequestHeaders:
      Host: bookstore-assets.s3.amazonaws.com
```

---

## Quick Reference Card

| Task | Native Gateway API | Traefik Middleware |
|------|-------------------|-------------------|
| Path matching | ✅ PathPrefix, Exact, RegEx | N/A |
| URL rewriting | ✅ URLRewrite | 🔧 StripPrefix (simpler) |
| Redirects | ✅ RequestRedirect (basic) | 🔧 RedirectRegex (advanced) |
| Set request headers | ✅ RequestHeaderModifier | 🔧 Headers (more options) |
| Set response headers | ✅ ResponseHeaderModifier | 🔧 Headers (easier) |
| Rate limiting | ➖ Not available | 🔧 RateLimit |
| Authentication | ➖ Not available | 🔧 BasicAuth, ForwardAuth |
| Error pages | ➖ Not available | 🔧 Errors |
| Compression | ➖ Not available | 🔧 Compress |
| Traffic split | ✅ backendRefs weights | N/A |
| Circuit breaker | ➖ Not available | 🔧 CircuitBreaker |

---

## Additional Resources

- **Gateway API:** <https://gateway-api.sigs.k8s.io/>
- **Traefik Gateway API:** <https://doc.traefik.io/traefik/routing/providers/kubernetes-gateway/>
- **Traefik Middleware:** <https://doc.traefik.io/traefik/middlewares/overview/>
- **HTTPRoute Spec:** <https://gateway-api.sigs.k8s.io/api-types/httproute/>

---

**Author:** Aleksandro Matejic  
**Example App:** Online Bookstore  
**Date:** January 2026  
**Version:** 1.0
