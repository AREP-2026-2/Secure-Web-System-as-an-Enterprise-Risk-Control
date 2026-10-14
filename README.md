# Secure Web System as an Enterprise Risk Control

React SPA authenticated with **Auth0**, calling a **Spring Boot** OAuth 2.0 Resource Server,
published over **HTTPS** through **Apache** with a **Let's Encrypt** certificate and a **Duck DNS** hostname on AWS EC2.

**Deployed URL:** https://YOUR-SUBDOMAIN.duckdns.org

## MADE BY:
- Sebastian Albarracin Silva
 
## Architecture

![Architecture](docs/architecture.png)

- The browser logs in against Auth0 and receives an access token for the audience `https://enterprise-risk-api`.
- The browser connects only on port 443 to Apache, which serves the React build and forwards `/api/` to Spring Boot on `127.0.0.1:8080`.
- Spring Boot validates the JWT signature (Auth0 JWKS), issuer and audience.
- Certbot issues and renews the certificate; Duck DNS points the hostname to the EC2 public IP.

## Repository layout

| Path | Content |
|------|---------|
| `backend/` | Spring Boot API: `GET /api/risk-status` (protected), `GET /actuator/health` (public) |
| `frontend/` | Vite + React + `@auth0/auth0-react` |
| `deploy/` | systemd unit, Apache virtual host, Duck DNS update script template |
| `docs/` | Architecture diagram and evidence screenshots |

## Run locally

```bash
# backend (http://localhost:8080)
cd backend
./mvnw spring-boot:run      # Windows: mvnw.cmd spring-boot:run

# frontend (http://localhost:5173)
cd frontend
cp .env.example .env.local  # fill in your Auth0 values
npm install
npm run dev
```

Before running the backend, set your tenant in `backend/src/main/resources/application.properties`
or export `SPRING_SECURITY_OAUTH2_RESOURCESERVER_JWT_ISSUER_URI=https://YOUR_TENANT.us.auth0.com/`.

`curl -i http://localhost:8080/api/risk-status` without a token returns `401 Unauthorized`.

## Evidence

| Evidence | File |
|----------|------|
| Authenticated API call | `docs/authenticated-call.png` |
| Valid certificate | `docs/certificate.png` |

## Risk-control reflection

| Control | Risk reduced | Evidence |
|---------|--------------|----------|
| Auth0 authentication | Impersonation and weak credential handling | Auth0 login flow and user identity |
| JWT validation at the API | Unauthorized API access | 401 without a valid token |
| Audience validation | Token reuse against another API | API accepts only its intended audience |
| HTTPS with a valid certificate | Credential or token exposure in transit | Browser certificate details |
| Apache reverse proxy | Direct exposure of the application port | Only ports 80 and 443 are public; API bound to 127.0.0.1 |
| Central service logs | Delayed incident investigation | `journalctl -u enterprise-risk-api` |

**Authentication vs. authorization:** authentication proves *who* the user is (Auth0 login).
Authorization decides *what* that identity may do (the API only accepts tokens issued by our tenant for our audience).
