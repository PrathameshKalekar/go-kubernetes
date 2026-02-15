# Ride Sharing – Go + Kubernetes

Ride-sharing app with Go microservices, Kubernetes, and Tilt.

## Prerequisites (Mac & Windows)

- **Docker Desktop** – with Kubernetes enabled
- **Tilt** – [tilt.dev](https://docs.tilt.dev/install.html)
  - Mac: `brew install tilt-dev/tap/tilt`
  - Windows: `curl -fsSL https://raw.githubusercontent.com/tilt-dev/tilt/master/scripts/install.ps1 | powershell -`
- **Go** 1.23+
- **MongoDB Atlas** – free cluster at [cloud.mongodb.com](https://cloud.mongodb.com)
- **Stripe** – test keys from [dashboard.stripe.com/apikeys](https://dashboard.stripe.com/apikeys)

### Docker Desktop

1. **Settings** → **Kubernetes** → enable **Kubernetes** → **Apply & Restart**

---

## Steps to Run

### 1. Create secrets

```bash
cp infra/development/k8s/secrets.template.yaml infra/development/k8s/secrets.yaml
```

### 2. Edit `secrets.yaml`

| Key | Where |
|-----|-------|
| stripe-secret-key | Stripe Dashboard → API Keys → Secret key |
| stripe-webhook-key | Run `stripe listen` (see below) and copy `whsec_...` |
| mongodb uri | MongoDB Atlas → Connect → Copy connection string |

Encode special characters in passwords (e.g. `@` → `%40`).

### 3. Set architecture in Tiltfile (line 5)

- **Mac Apple Silicon (M1/M2/M3):** `go_arch = 'arm64'`
- **Mac Intel / Windows:** `go_arch = 'amd64'`

### 4. Run

```bash
tilt up
```

### 5. Access

- **App:** http://localhost:3000
- **RabbitMQ:** http://localhost:15672 (guest / guest)

---

## Payments (optional)

Install Stripe CLI: [stripe.com/docs/stripe-cli](https://stripe.com/docs/stripe-cli) (Mac: `brew install stripe/stripe-cli/stripe`)

In a separate terminal:

```bash
stripe listen --forward-to localhost:8081/webhook/stripe
```

Copy the `whsec_...` value into `secrets.yaml` → `stripe-webhook-key`, then:

```bash
kubectl apply -f infra/development/k8s/secrets.yaml
kubectl rollout restart deployment api-gateway
```

---

## Troubleshooting

| Error | Fix |
|-------|-----|
| Docker daemon | Start Docker Desktop |
| Kubernetes connection refused | Enable Kubernetes in Docker Desktop |
| Exec format error | Set `go_arch` in Tiltfile (arm64 for Apple Silicon, amd64 for Intel/Windows) |
| MongoDB unescaped @ | URL-encode `@` as `%40` in the password |
