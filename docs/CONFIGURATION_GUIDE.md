# BPN Discovery Configuration Guide

This guide explains how to configure BPN Discovery with external Keycloak authentication and PostgreSQL database.

## Конфигурация с внешним Keycloak и PostgreSQL / Configuration with External Keycloak and PostgreSQL

### Русский / Russian

#### Обзор

BPN Discovery можно настроить для работы с:
- Внешней базой данных PostgreSQL (вместо встроенной)
- Keycloak для OAuth2 аутентификации

#### Предварительные требования

1. **Keycloak сервер** с настроенным realm
   - URL issuer (например: `https://centralidp.ippcp-dev.epa.si/auth/realms/CX-Central`)
   - Настроенный клиент для BPN Discovery
   - Client ID и Client Secret

2. **PostgreSQL база данных**
   - PostgreSQL 12+
   - Созданная база данных (например: `bpndiscovery`)
   - Пользователь с правами доступа к базе данных

3. **Kubernetes кластер**
   - Kubernetes 1.19+
   - Helm 3.2.0+
   - Ingress controller (например: nginx-ingress)

#### Быстрый старт

1. **Подготовьте базу данных PostgreSQL:**
```sql
CREATE DATABASE bpndiscovery;
CREATE USER bpndiscovery WITH ENCRYPTED PASSWORD 'your-secure-password';
GRANT ALL PRIVILEGES ON DATABASE bpndiscovery TO bpndiscovery;
```

2. **Настройте клиента в Keycloak:**
   - Создайте нового клиента в вашем realm
   - Установите Client ID (например: `bpndiscovery-client`)
   - Включите Client authentication
   - Установите Valid Redirect URIs
   - Скопируйте Client Secret

3. **Создайте файл конфигурации values.yaml:**

```yaml
# Отключить встроенную PostgreSQL
enablePostgres: false

bpndiscovery:
  # Включить OAuth2 аутентификацию
  authentication: true
  
  # Настройки Keycloak/IDP
  idp:
    # URL issuer вашего Keycloak realm
    issuerUri: "https://centralidp.ippcp-dev.epa.si/auth/realms/CX-Central"
    # Client ID для BPN Discovery
    publicClientId: "bpndiscovery-client"
    # Имя claim в JWT токене, содержащего BPN
    bpnIdClaimName: "bpn"
  
  # Конфигурация внешней базы данных PostgreSQL
  dataSource:
    driverClassName: org.postgresql.Driver
    sqlInitPlatform: pg
    # URL подключения к вашей базе данных PostgreSQL
    url: jdbc:postgresql://172.26.4.186:5432/bpndiscovery
    # Имя пользователя базы данных
    user: bpndiscovery
    # Пароль базы данных
    password: your-secure-password
  
  # Настройки endpoint для саморегистрации на discovery-finder
  bpndiscoveryEndpoint:
    allowedTypes: oen,wmi,bpid
    description: "BPN Discovery Service"
    endpointAddress: "https://your-domain.com/bpndiscovery"
    documentation: "https://your-domain.com/bpndiscovery/swagger/index.html"
    timeToLive: "31536000"
  
  # Настройки клиента Discovery Finder
  discoveryfinderClient:
    baseUrl: "https://discovery-finder.example.com"
    registration:
      clientId: "bpndiscovery-client"
      clientSecret: "your-client-secret"
      authorizationGrantType: client_credentials
    schedulerCronFrequency: "0 0 0 * * *"
    provider:
      tokenUri: "https://centralidp.ippcp-dev.epa.si/auth/realms/CX-Central/protocol/openid-connect/token"
  
  # Настройки Ingress
  ingress:
    enabled: true
    className: nginx
    tls: true
    urlPrefix: /bpndiscovery
  
  host: your-domain.com
  
  resources:
    requests:
      memory: "512Mi"
      cpu: "250m"
    limits:
      memory: "1024Mi"
      cpu: "750m"

# Отключить встроенную PostgreSQL
postgresql:
  enabled: false
```

4. **Установите BPN Discovery с помощью Helm:**

```bash
# Обновить зависимости
helm dep up charts/bpndiscovery

# Создать namespace
kubectl create namespace discovery

# Установить с вашим файлом конфигурации
helm install bpndiscovery charts/bpndiscovery -n discovery -f your-values.yaml
```

#### Важные параметры конфигурации

| Параметр | Описание | Пример |
|----------|----------|---------|
| `enablePostgres` | Отключить встроенную PostgreSQL | `false` |
| `bpndiscovery.authentication` | Включить OAuth2 аутентификацию | `true` |
| `bpndiscovery.idp.issuerUri` | URL issuer Keycloak realm | `https://keycloak.example.com/auth/realms/CX-Central` |
| `bpndiscovery.idp.publicClientId` | Client ID в Keycloak | `bpndiscovery-client` |
| `bpndiscovery.dataSource.url` | JDBC URL базы данных | `jdbc:postgresql://host:5432/database` |
| `bpndiscovery.dataSource.user` | Пользователь БД | `bpndiscovery` |
| `bpndiscovery.dataSource.password` | Пароль БД | `your-password` |

#### Безопасность

**Важно:** Никогда не храните пароли в открытом виде в values.yaml!

Используйте Kubernetes Secrets для хранения конфиденциальных данных:

```bash
# Создать secret для базы данных
kubectl create secret generic bpndiscovery-db-secret \
  --from-literal=password='your-db-password' \
  -n discovery

# Создать secret для Keycloak client
kubectl create secret generic bpndiscovery-keycloak-secret \
  --from-literal=client-secret='your-client-secret' \
  -n discovery
```

Затем обновите values.yaml для использования secrets:

```yaml
bpndiscovery:
  dataSource:
    password: "" # Оставить пустым
    # Добавить в deployment.yaml ссылку на secret
  
  discoveryfinderClient:
    registration:
      clientSecret: "" # Оставить пустым
      # Добавить в deployment.yaml ссылку на secret
```

---

### English

#### Overview

BPN Discovery can be configured to work with:
- External PostgreSQL database (instead of the bundled one)
- Keycloak for OAuth2 authentication

#### Prerequisites

1. **Keycloak server** with configured realm
   - Issuer URL (e.g., `https://centralidp.ippcp-dev.epa.si/auth/realms/CX-Central`)
   - Configured client for BPN Discovery
   - Client ID and Client Secret

2. **PostgreSQL database**
   - PostgreSQL 12+
   - Created database (e.g., `bpndiscovery`)
   - User with database access rights

3. **Kubernetes cluster**
   - Kubernetes 1.19+
   - Helm 3.2.0+
   - Ingress controller (e.g., nginx-ingress)

#### Quick Start

1. **Prepare PostgreSQL database:**
```sql
CREATE DATABASE bpndiscovery;
CREATE USER bpndiscovery WITH ENCRYPTED PASSWORD 'your-secure-password';
GRANT ALL PRIVILEGES ON DATABASE bpndiscovery TO bpndiscovery;
```

2. **Configure client in Keycloak:**
   - Create a new client in your realm
   - Set Client ID (e.g., `bpndiscovery-client`)
   - Enable Client authentication
   - Set Valid Redirect URIs
   - Copy Client Secret

3. **Create values.yaml configuration file:**

See the complete example in the Russian section above.

4. **Install BPN Discovery using Helm:**

```bash
# Update dependencies
helm dep up charts/bpndiscovery

# Create namespace
kubectl create namespace discovery

# Install with your configuration file
helm install bpndiscovery charts/bpndiscovery -n discovery -f your-values.yaml
```

#### Important Configuration Parameters

| Parameter | Description | Example |
|-----------|-------------|---------|
| `enablePostgres` | Disable bundled PostgreSQL | `false` |
| `bpndiscovery.authentication` | Enable OAuth2 authentication | `true` |
| `bpndiscovery.idp.issuerUri` | Keycloak realm issuer URL | `https://keycloak.example.com/auth/realms/CX-Central` |
| `bpndiscovery.idp.publicClientId` | Client ID in Keycloak | `bpndiscovery-client` |
| `bpndiscovery.dataSource.url` | Database JDBC URL | `jdbc:postgresql://host:5432/database` |
| `bpndiscovery.dataSource.user` | Database user | `bpndiscovery` |
| `bpndiscovery.dataSource.password` | Database password | `your-password` |

#### Security

**Important:** Never store passwords in plain text in values.yaml!

Use Kubernetes Secrets to store sensitive data:

```bash
# Create secret for database
kubectl create secret generic bpndiscovery-db-secret \
  --from-literal=password='your-db-password' \
  -n discovery

# Create secret for Keycloak client
kubectl create secret generic bpndiscovery-keycloak-secret \
  --from-literal=client-secret='your-client-secret' \
  -n discovery
```

Then update values.yaml to use secrets:

```yaml
bpndiscovery:
  dataSource:
    password: "" # Leave empty
    # Add reference to secret in deployment.yaml
  
  discoveryfinderClient:
    registration:
      clientSecret: "" # Leave empty
      # Add reference to secret in deployment.yaml
```

#### Verification

After installation, verify the deployment:

```bash
# Check pods status
kubectl get pods -n discovery

# Check BPN Discovery logs
kubectl logs -n discovery deployment/bpndiscovery -f

# Test the API endpoint
curl https://your-domain.com/bpndiscovery/api/v1.0/administration/connectors/discovery/search
```

#### Troubleshooting

**Database connection issues:**
- Verify PostgreSQL is accessible from the Kubernetes cluster
- Check database credentials
- Ensure database user has proper permissions

**Authentication issues:**
- Verify Keycloak issuer URI is correct
- Check client ID and secret
- Ensure the BPN claim is correctly configured in Keycloak

**Ingress issues:**
- Verify ingress controller is running
- Check ingress configuration and annotations
- Ensure DNS is configured correctly

For more help, check the [main documentation](../../README.md) or [installation guide](../../INSTALL.md).

## Complete Example Based on BPDM Configuration

If you're migrating from BPDM or want a similar configuration, here's an example that matches the BPDM configuration pattern:

```yaml
# Similar to BPDM configuration structure
enablePostgres: false

bpndiscovery:
  authentication: true
  
  # Keycloak configuration
  idp:
    issuerUri: "https://centralidp.ippcp-dev.epa.si/auth/realms/CX-Central"
    publicClientId: "Cl16-CX-BPNDiscovery"
    bpnIdClaimName: "bpn"
  
  # External PostgreSQL
  dataSource:
    driverClassName: org.postgresql.Driver
    sqlInitPlatform: pg
    url: jdbc:postgresql://172.26.4.186:5432/bpndiscovery
    user: bpndiscovery
    password: your-secure-password-here
  
  # Discovery Finder integration
  discoveryfinderClient:
    baseUrl: "http://discovery-finder.tx-core.svc.cluster.local"
    registration:
      clientId: "Cl16-CX-BPNDiscovery"
      clientSecret: "your-client-secret-here"
      authorizationGrantType: client_credentials
    provider:
      tokenUri: "https://centralidp.ippcp-dev.epa.si/auth/realms/CX-Central/protocol/openid-connect/token"
  
  # Public endpoint configuration
  bpndiscoveryEndpoint:
    allowedTypes: oen,wmi,bpid
    description: "BPN Discovery Service"
    endpointAddress: "https://bpndiscovery.example.com/bpndiscovery"
    documentation: "https://bpndiscovery.example.com/bpndiscovery/swagger/index.html"
    timeToLive: "31536000"
  
  # Resource configuration
  resources:
    requests:
      memory: "512Mi"
      cpu: "250m"
    limits:
      memory: "1Gi"
      cpu: "1"

postgresql:
  enabled: false
```

This configuration:
- Uses the same Keycloak instance as BPDM services
- Connects to the same PostgreSQL server (but different database)
- Uses similar client credentials pattern
- Disables bundled PostgreSQL
- Follows the same resource allocation pattern as BPDM
