# Understanding YAML Spaces for Kubernetes Service

This document explains the correct **indentation and spaces** for a Kubernetes Service YAML.

---

## Example: Service YAML

```yaml
apiVersion: v1
kind: Service
metadata:
  name: movie-app-service
spec:
  type: LoadBalancer
  selector:
    app: movie-app
  ports:
    - port: 80
      targetPort: 5000
```

---

## Space Map (Line by Line)

```
1 2 3 4 5 6 7 8 9 10 11 12
| | | | | | | | | | | |
apiVersion: v1       # 0 spaces (root)
kind: Service         # 0 spaces (root)
metadata:             # 0 spaces (root)
  name: movie-app-service   # 2 spaces under metadata
spec:                 # 0 spaces (root)
  type: LoadBalancer  # 2 spaces under spec
  selector:           # 2 spaces under spec
    app: movie-app    # 4 spaces under root (2 under selector)
  ports:              # 2 spaces under spec
    - port: 80       # 4 spaces under root (list item)
      targetPort: 5000  # 6 spaces under root (2 more under - port)
```

---

## Rules for YAML Indentation

1. **Use 2 spaces per level** (Kubernetes standard).  
2. **Never use tabs** — YAML does not allow tabs.  
3. **List items** (`-`) must be indented 2 spaces more than the parent key.  
4. **Nested keys under list items** get 2 extra spaces beyond the list item.  
5. **No extra spaces before colons**:

✅ Correct:  
```yaml
port: 80
```

❌ Wrong:  
```yaml
port : 80  # extra space before colon
```

---

## Common Mistakes

- Misaligned nested keys (`targetPort` not indented correctly)  
- Mixing tabs and spaces  
- Extra spaces after key names or colons  

---

## Summary

- Root keys: `apiVersion`, `kind`, `metadata`, `spec` → 0 spaces  
- Nested under a key → +2 spaces  
- List items → +2 spaces from parent  
- Nested under list item → +2 spaces more  

Following this structure will prevent YAML parsing errors in Kubernetes manifests.
