# Quick Reference: BPN Discovery Configuration (Similar to BPDM)

## Important Note / Важное примечание

**BPN Discovery ≠ BPDM Services**

BPN Discovery is a **standalone service** and does NOT connect to BPDM services (orchestrator, pool, gate). 

**BPN Discovery - это самостоятельный сервис**, который НЕ подключается к сервисам BPDM (orchestrator, pool, gate).

### Architecture Difference / Архитектурная разница:

**BPDM Services (взаимосвязанные):**
```
bpdm-gate ← → bpdm-orchestrator ← → bpdm-pool
```

**BPN Discovery (независимый):**
```
BPN Discovery → Keycloak (authentication)
BPN Discovery → PostgreSQL (data storage)
BPN Discovery → Discovery Finder (optional registration)
```

BPN Discovery does not require connections to BPDM orchestrator, pool, or gate.

BPN Discovery не требует подключений к BPDM orchestrator, pool или gate.

## Сравнение конфигурации BPDM и BPN Discovery / BPDM vs BPN Discovery Configuration Comparison

### Русский

Если вы настраивали BPDM сервисы и хотите настроить BPN Discovery аналогичным образом, используйте следующую конфигурацию:

#### BPDM конфигурация (для справки):
```yaml
bpdm-gate:
  enabled: true
  applicationSecrets:
    spring:
      datasource:
        url: jdbc:postgresql://172.26.4.186:5432/bpdm
        username: bpdm
        password: your-secure-password-here
  applicationConfig:
    bpdm:
      security:
        auth-server-url: https://centralidp.ippcp-dev.epa.si/auth
  postgres:
    enabled: false
  centralidp:
    enabled: false
```

#### Эквивалентная конфигурация BPN Discovery:
```yaml
# values.yaml для BPN Discovery
enablePostgres: false

bpndiscovery:
  authentication: true
  
  # Настройки базы данных (аналог applicationSecrets.spring.datasource)
  dataSource:
    driverClassName: org.postgresql.Driver
    sqlInitPlatform: pg
    url: jdbc:postgresql://172.26.4.186:5432/bpndiscovery
    user: bpndiscovery
    password: your-secure-password-here
  
  # Настройки Keycloak (аналог applicationConfig.bpdm.security)
  idp:
    issuerUri: "https://centralidp.ippcp-dev.epa.si/auth/realms/CX-Central"
    publicClientId: "Cl16-CX-BPNDiscovery"
    bpnIdClaimName: "bpn"
  
  # Настройки Discovery Finder (если используется)
  discoveryfinderClient:
    baseUrl: "http://discovery-finder.tx-core.svc.cluster.local"
    registration:
      clientId: "Cl16-CX-BPNDiscovery"
      clientSecret: "your-client-secret-here"
      authorizationGrantType: client_credentials
    provider:
      tokenUri: "https://centralidp.ippcp-dev.epa.si/auth/realms/CX-Central/protocol/openid-connect/token"
  
  # Публичный endpoint
  bpndiscoveryEndpoint:
    allowedTypes: oen,wmi,bpid
    description: "BPN Discovery Service"
    endpointAddress: "https://your-domain.com/bpndiscovery"
    documentation: "https://your-domain.com/bpndiscovery/swagger/index.html"
    timeToLive: "31536000"
  
  # Ресурсы (аналог resources в BPDM)
  resources:
    requests:
      memory: "512Mi"
      cpu: "250m"
    limits:
      memory: "1Gi"
      cpu: "1"

# Отключить встроенную PostgreSQL (аналог postgres.enabled: false)
postgresql:
  enabled: false
```

### Ключевые отличия / Key Differences:

| BPDM | BPN Discovery | Назначение / Purpose |
|------|---------------|----------------------|
| `applicationSecrets.spring.datasource` | `bpndiscovery.dataSource` | Настройки подключения к БД / DB connection settings |
| `applicationConfig.bpdm.security.auth-server-url` | `bpndiscovery.idp.issuerUri` | URL Keycloak сервера / Keycloak server URL |
| `postgres.enabled: false` | `enablePostgres: false` | Отключение встроенной БД / Disable bundled DB |
| `centralidp.enabled: false` | Не требуется / Not needed | BPN Discovery не имеет встроенного IDP / BPN Discovery has no bundled IDP |
| Связи с orchestrator/pool/gate | **НЕ требуется** / **NOT required** | BPN Discovery - независимый сервис / BPN Discovery is standalone |

### Роли в Keycloak / Keycloak Roles:

BPN Discovery требует следующие роли для пользователей/клиентов:

**Required Roles:**
- `view_bpn_discovery` - Для просмотра и поиска BPN данных / For viewing and searching BPN data
  - Эндпоинты / Endpoints: GET /**, POST /api/.../search
  
- `add_bpn_discovery` - Для добавления BPN данных / For adding BPN data
  - Эндпоинты / Endpoints: POST /api/.../bpnDiscovery, POST /api/.../batch
  
- `delete_bpn_discovery` - Для удаления BPN данных / For deleting BPN data
  - Эндпоинты / Endpoints: DELETE /api/.../bpnDiscovery/**

**Настройка ролей в Keycloak:**
1. Откройте Keycloak Admin Console
2. Перейдите в realm (например: CX-Central)
3. Clients → выберите клиент → Roles
4. Создайте три роли выше
5. Назначьте роли пользователям или сервисным аккаунтам

**Configuring Roles in Keycloak:**
1. Open Keycloak Admin Console
2. Navigate to your realm (e.g., CX-Central)
3. Clients → select your client → Roles
4. Create the three roles above
5. Assign roles to users or service accounts

### English

If you've been configuring BPDM services and want to configure BPN Discovery similarly, use the following configuration:

#### BPDM Configuration (for reference):
See above in the Russian section.

#### Equivalent BPN Discovery Configuration:
See above in the Russian section.

### Installation Command / Команда установки:

```bash
# Создать namespace / Create namespace
kubectl create namespace discovery

# Установить с вашей конфигурацией / Install with your configuration
helm install bpndiscovery charts/bpndiscovery -n discovery \
  -f charts/bpndiscovery/values-external-keycloak-postgres.yaml \
  --set bpndiscovery.dataSource.url=jdbc:postgresql://172.26.4.186:5432/bpndiscovery \
  --set bpndiscovery.dataSource.user=bpndiscovery \
  --set bpndiscovery.dataSource.password=your-secure-password-here \
  --set bpndiscovery.idp.issuerUri=https://centralidp.ippcp-dev.epa.si/auth/realms/CX-Central \
  --set bpndiscovery.idp.publicClientId=Cl16-CX-BPNDiscovery
```

### Checklist for Migration from BPDM Configuration:

- [ ] Create database for BPN Discovery (can be on the same PostgreSQL server as BPDM)
- [ ] Create Keycloak client for BPN Discovery (can use the same Keycloak realm as BPDM)
- [ ] Update database connection URL, username, password
- [ ] Update Keycloak issuerUri (same as BPDM if using same realm)
- [ ] Update client ID and secret for BPN Discovery
- [ ] Set `enablePostgres: false` to disable bundled database
- [ ] Configure ingress and public endpoints
- [ ] Install using Helm with custom values

For more detailed information, see:
- [Complete Configuration Guide](CONFIGURATION_GUIDE.md)
- [Example Values File](../charts/bpndiscovery/values-external-keycloak-postgres.yaml)
- [Installation Guide](../INSTALL.md)
