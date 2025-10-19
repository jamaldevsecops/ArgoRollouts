# Canary Deployment Theory

## 🐦 Overview

**Canary deployment** is a progressive delivery strategy where a new version of an application is gradually rolled out to a small subset of users before being released to everyone.  
This approach helps identify bugs, performance issues, or user experience problems early—without impacting the majority of users.

---

## ⚙️ Workflow

1. **Deploy the new version (Canary):**  
   The new version (say `v2`) is deployed alongside the current stable version (`v1`). Both versions are running in the cluster.

2. **Route a small percentage of traffic to the Canary:**  
   Only a small portion (e.g., 5–10%) of user traffic is sent to the Canary while most traffic continues to go to the stable version.

3. **Monitor metrics and logs:**  
   During this phase, you monitor key metrics like latency, error rates, and resource usage to ensure the new version behaves as expected.

4. **Gradually increase traffic:**  
   If the new version performs well, more traffic is shifted to the Canary until it eventually serves 100% of traffic.

5. **Promote or Rollback:**  
   - If the deployment is successful, promote the Canary to become the new stable version.  
   - If problems occur, rollback to the previous stable version.

---

## 📈 Benefits

✅ Safer deployments — reduces the risk of downtime.  
✅ Real user testing — exposes new version to a limited audience first.  
✅ Continuous delivery friendly — fits CI/CD pipelines seamlessly.  
✅ Quick rollback — revert instantly if issues arise.

---

## ⚠️ Drawbacks

❌ Slightly more complex setup — needs traffic routing logic (usually via ingress/gateway).  
❌ Resource overhead — both versions run simultaneously.  
❌ Metrics dependency — success heavily depends on monitoring systems.

---

## 🚀 Canary Deployment in Argo Rollouts

Argo Rollouts enhances the canary strategy using Kubernetes-native custom resources.

**Key Features:**
- Gradual traffic shifting (e.g., 5%, 25%, 50%, 100%)  
- Automated analysis (integration with Prometheus, Kayenta, etc.)  
- Manual or automated promotion  
- Rollback support  

Example (simplified):
```yaml
strategy:
  canary:
    steps:
    - setWeight: 20
    - pause: { duration: 1m }
    - setWeight: 50
    - pause: { duration: 2m }
```

This configuration sends 20% of traffic to the new version, waits 1 minute, then increases to 50%, and so on.

---

## 🌍 Real-World Example

| Step | Traffic to Old (Stable) | Traffic to New (Canary) | Action |
|------|--------------------------|--------------------------|--------|
| 1 | 100% | 0% | Start rollout |
| 2 | 95% | 5% | Initial canary testing |
| 3 | 70% | 30% | Monitor and analyze |
| 4 | 0% | 100% | Promote if stable |
| 5 | 100% | 0% | Rollback if failure |

---

## 🧩 Summary

Canary deployment enables **controlled, data-driven releases** that improve reliability and confidence.  
With **Argo Rollouts**, this process becomes automated, observable, and Kubernetes-native—allowing safe progressive delivery.
