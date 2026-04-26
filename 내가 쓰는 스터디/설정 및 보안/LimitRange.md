## 목차

- [1장 LimitRange 란?](#1장-limitrange-란)
  - [✅ 1. LimitRange](#-1-limitrange)
  - [✅ 2. YAML 예시](#-2-yaml-예시)

---

# 1장 LimitRange 란?

## ✅ 1. LimitRange

✔ 한줄 정리

```
👉 Pod / Container 단위 리소스 제한 및 기본값 설정
```

✔ 역할

```
Pod 하나당 제한
기본값 자동 설정
```

## ✅ 2. YAML 예시

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: dev-limit
  namespace: dev
spec:
  limits:
    - type: Container # Container/Pod 중 선택
      default: # limit 기본값
        cpu: "500m"
        memory: "512Mi"
      defaultRequest: # request 기본값
        cpu: "200m"
        memory: "256Mi"
      max: # 최대 허용 범위
        cpu: "1"
        memory: "1Gi"
      min: # 최소 허용범위
        cpu: "100m"
        memory: "128Mi"
```
