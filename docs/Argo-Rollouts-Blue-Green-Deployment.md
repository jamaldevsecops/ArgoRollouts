# 🌀 Argo Rollouts Blue-Green Deployment

## 📘 Overview
**Blue-Green Deployment** allows you to switch traffic between two versions of your app (Blue and Green) **instantly and safely** using **Argo Rollouts**.

---

## 🏗️ 1. Namespace Setup

```bash
kubectl create namespace demo-bluegreen
kubectl config set-context --current --namespace=demo-bluegreen
```

---

## 🚀 2. Rollout Definition

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: rollouts-demo
  namespace: demo-bluegreen
spec:
  replicas: 3
  strategy:
    blueGreen:
      activeService: rollouts-demo-active
      previewService: rollouts-demo-preview
      autoPromotionEnabled: false   # manual promotion for safety
  selector:
    matchLabels:
      app: rollouts-demo
  template:
    metadata:
      labels:
        app: rollouts-demo
    spec:
      containers:
      - name: rollouts-demo
        image: argoproj/rollouts-demo:blue   # start with blue version
        ports:
        - containerPort: 8080
```

Apply it:
```bash
kubectl apply -f rollout.yaml
```

---

## 🌐 3. Service Definitions

### Active Service (Production Traffic)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: rollouts-demo-active
spec:
  ports:
  - port: 80
    targetPort: 8080
  selector:
    app: rollouts-demo
```

### Preview Service (Testing)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: rollouts-demo-preview
spec:
  ports:
  - port: 80
    targetPort: 8080
  selector:
    app: rollouts-demo
```

Apply:
```bash
kubectl apply -f services.yaml
```

---

## 🌍 4. Ingress Resources

### Ingress for Live Traffic
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: rollouts-demo-live-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
spec:
  rules:
  - host: rollouts-demo.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: rollouts-demo-active
            port:
              number: 80
```

### Ingress for Preview (Testing)
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: rollouts-demo-preview-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
spec:
  rules:
  - host: preview.rollouts-demo.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: rollouts-demo-preview
            port:
              number: 80
```

Apply:
```bash
kubectl apply -f ingress.yaml
```

---

## 🔄 5. Update to Green Version

Edit your rollout:
```bash
kubectl argo rollouts set image rollouts-demo rollouts-demo=argoproj/rollouts-demo:green
```

Watch progress:
```bash
kubectl argo rollouts get rollout rollouts-demo --watch
```

Once verified, **promote manually**:
```bash
kubectl argo rollouts promote rollouts-demo
```

Rollback (if needed):
```bash
kubectl argo rollouts undo rollouts-demo
```

---

## 📊 6. Verify Setup

Check rollout status:
```bash
kubectl argo rollouts get rollout rollouts-demo
```

Open dashboard:
```bash
kubectl argo rollouts dashboard
```

---

## 🔍 7. Traffic Flow Diagram

```
            ┌─────────────────────────────┐
            │        Ingress              │
            └────────────┬────────────────┘
                         │
           ┌─────────────┴─────────────┐
           │                           │
 ┌────────────────────────┐  ┌────────────────────────┐
 │ rollouts-demo-active   │  │ rollouts-demo-preview  │
 └────────┬───────────────┘  └────────┬───────────────┘
          │                           │
    ┌────────────┐              ┌────────────┐
    │ Blue Pods  │              │ Green Pods │
    └────────────┘              └────────────┘
```

---

## ✅ 8. Best Practices
- Always define **readiness probes** to prevent traffic to unhealthy pods.
- Keep **manual promotion** for production environments.
- Expose **preview ingress** only internally for QA/testing.
- Enable **auto rollback** or alerts based on metrics (optional).

---

## 🧩 Summary

| Resource | Purpose |
|-----------|----------|
| Rollout | Controls deployment logic |
| rollouts-demo-active | Production service |
| rollouts-demo-preview | Test service |
| rollouts-demo-live-ingress | User-facing ingress |
| rollouts-demo-preview-ingress | QA ingress |
| argoproj/rollouts-demo:blue | Blue image |
| argoproj/rollouts-demo:green | Green image |
