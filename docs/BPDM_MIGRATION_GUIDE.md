# Quick Reference: BPN Discovery Configuration (Similar to BPDM)

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
        password: UvWoXg7t9O2vegWWciOA
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
    password: UvWoXg7t9O2vegWWciOA
  
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
      clientSecret: "Lwrns3AzorK85Y5ZW9mYupTWLLYMNiaa"
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
  --set bpndiscovery.dataSource.password=UvWoXg7t9O2vegWWciOA \
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
