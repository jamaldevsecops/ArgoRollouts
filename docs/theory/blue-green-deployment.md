# 🟦🟩 Blue-Green Deployment — Theory

## 🔹 What is Blue-Green Deployment?
**Blue-Green Deployment** is a release management strategy used to minimize downtime and reduce risk when deploying new application versions.

It involves running **two identical environments** called:
- **Blue (Current Active)** — the version currently serving live traffic.
- **Green (New Version)** — the new version being deployed and tested.

At any time, only one of these environments is live (receiving user traffic).

---

## 🔄 How Blue-Green Deployment Works

1. **Current State (Blue Active):**
   - The `blue` version is deployed and actively serving user traffic.
   - The `green` environment is idle or used for staging.

2. **Deploy Green Version:**
   - A new version of the application (e.g., `v2`) is deployed to the `green` environment.
   - Both environments are running, but only **blue** serves production traffic.

3. **Testing & Validation:**
   - The `green` environment is validated internally (functional tests, smoke tests, etc.).
   - Since `green` is not serving live traffic yet, it can be safely tested.

4. **Switch Traffic:**
   - Once the `green` version is validated, the traffic is switched from **blue → green**.
   - The `green` environment becomes live, and the `blue` environment becomes idle.

5. **Rollback if Needed:**
   - If any issue arises in `green`, traffic can be instantly redirected back to `blue`.
   - This enables a **zero-downtime rollback**.

---

## ⚙️ Example Scenario

| Stage | Blue (v1) | Green (v2) | Traffic |
|--------|-----------|------------|----------|
| Initial | ✅ Active | ❌ Idle | 100% Blue |
| Deploy Green | ✅ Active | ⚙️ Deployed | 100% Blue |
| Test Green | ✅ Active | 🧪 Testing | 100% Blue |
| Promote Green | ❌ Idle | ✅ Active | 100% Green |
| Rollback (if needed) | ✅ Active | ❌ Idle | 100% Blue |

---

## 💡 Benefits

✅ **Zero downtime:** Traffic switches instantly between environments.  
✅ **Quick rollback:** Reverting to the previous version is nearly instant.  
✅ **Safe validation:** The new version can be fully tested before going live.  
✅ **Simplified releases:** Blue and Green environments are isolated.

---

## ⚠️ Drawbacks

⚠️ **Higher resource usage:** Two full environments are required (double infrastructure).  
⚠️ **Database complexity:** Schema or data migrations may require special handling.  
⚠️ **Traffic switch coordination:** Must ensure smooth DNS/service switching.

---

## 🚀 Blue-Green in Argo Rollouts

Argo Rollouts extends Kubernetes by providing **Blue-Green deployment natively** using a `Rollout` custom resource.

Example snippet:
```yaml
strategy:
  blueGreen:
    activeService: rollout-bluegreen-active
    previewService: rollout-bluegreen-preview
    autoPromotionEnabled: false
```

- **activeService** → routes traffic to the current live version (Blue).  
- **previewService** → used for testing the new version (Green).  
- **autoPromotionEnabled: false** → prevents automatic traffic switch until you promote manually.

You can then promote using:
```bash
kubectl argo rollouts promote rollout rollout-bluegreen
```

---

## 🧠 Summary

| Aspect | Description |
|---------|--------------|
| Purpose | Deploy new version safely with zero downtime |
| Key Components | Blue (current) & Green (new) environments |
| Managed by | Argo Rollouts Controller |
| Promotion | Manual or automatic |
| Rollback | Instant traffic switch to previous version |

---

### 📘 In short:
> Blue-Green Deployment = Two environments (Blue and Green), one live at a time, traffic instantly switchable for safe releases.
