# dockhand-stacks

`club012.com` 인프라를 구성하는 Docker Compose 스택 모음입니다. [Dockhand](https://github.com/fnsys/dockhand)로 컨테이너 스택을 관리하며, 각 서비스는 독립된 디렉토리와 `compose.yaml`로 분리되어 있습니다.

## 디렉토리 구조

```
.
├── compose.yaml              # Dockhand 관리 Web UI (3000)
├── auth/                      # 인증 및 디렉토리 서비스
│   ├── authentik/             # Authentik IdP (9000/9443) + PostgreSQL
│   ├── keycloak/              # Keycloak OIDC/SAML IdP (8080) + PostgreSQL
│   └── openldap/              # OpenLDAP (389/636) + phpLDAPadmin (8081)
│       └── v2/                # OpenLDAP v2 독립 구성
├── channel/                   # 커뮤니케이션 / 메신저
│   └── mattermost/            # Mattermost (8065, 통화 8443)
│                              # (공용 PostgreSQL 연동 또는 독립 PostgreSQL 선택)
├── cms/                       # 업무 관리 / 협업 도구
│   └── redmine/               # Redmine 프로젝트 관리 (3000) + MySQL (3306)
├── database/                  # 공용 데이터베이스 (공유 네트워크)
│   ├── mysql/                 # MySQL 8.4 (3306)
│   ├── postgres/              # PostgreSQL 18.4 (5432)
│   └── redis/                 # Redis 8.8 (6379)
└── secrets/                   # 시크릿 및 비밀번호 관리
    ├── vault/                 # HashiCorp Vault (8200) KMS / Transit / OIDC 연동
    └── vaultwarden/           # Vaultwarden Bitwarden 호환 서버 (8000)
```

## 서비스 상세 및 네트워크 구성

### 1. 관리 (Management)
- **Dockhand** (`/compose.yaml`): Docker 소켓(`/var/run/docker.sock`)과 볼륨을 마운트하여 컨테이너 스택을 관리하는 웹 UI 대시보드 (`3000`).

### 2. 인증 (Auth)
- **Authentik** (`auth/authentik/`): 올인원 오픈소스 IdP. 전용 PostgreSQL과 함께 구동 (`9000/9443`).
- **Keycloak** (`auth/keycloak/`): OIDC/OAuth2/SAML 인증 제공자 (`8080`). `devops` realm 설정 및 Vault 등 사내 시스템의 OIDC 프로바이더로 연동.
- **OpenLDAP** (`auth/openldap/`): 디렉토리 서비스 (`389/636`) 및 웹 기반 관리 툴 phpLDAPadmin (`8081`). `dc=club012,dc=com` 베이스 DN 사용.

### 3. 커뮤니케이션 (Channel)
- **Mattermost** (`channel/mattermost/`):
  - `compose.yaml`: 외부 공유 `postgres` 네트워크를 통해 `database/postgres`와 연동.
  - `compose-with-postgres.yaml`: 단독 실행용 내장 PostgreSQL 포함 구성.
  - Web/API 포트 (`8065`), 음성 통화 포트 (`8443/udp`, `8443/tcp`).

### 4. 업무 관리 (CMS)
- **Redmine** (`cms/redmine/`): 이슈 및 프로젝트 관리 시스템 (`3000`). 스택 내부에 자체 MySQL 8.0 컨테이너 포함.

### 5. 공용 데이터베이스 (Database)
외부 공유 네트워크(`external: true`)를 활용하여 타 스택과 격리 및 연동:
- **MySQL** (`database/mysql/`): MySQL 8.4, `mysql` 네트워크 (`3306`).
- **PostgreSQL** (`database/postgres/`): PostgreSQL 18.4, `postgres` 네트워크 (`5432`).
- **Redis** (`database/redis/`): Redis 8.8 인메모리 캐시/스토어 (`6379`).

> [!NOTE]
> 외부 공유 네트워크(`postgres`, `mysql`)를 사용하는 스택을 실행하기 전에 해당 네트워크가 Docker에 생성되어 있어야 합니다:
> ```bash
> docker network create postgres
> docker network create mysql
> ```

### 6. 시크릿 관리 (Secrets)
- **HashiCorp Vault** (`secrets/vault/`): 시크릿 저장소 및 Transit 암호화 엔진 (`8200`). Keycloak OIDC 연동 가이드 및 설정 스크립트 포함.
- **Vaultwarden** (`secrets/vaultwarden/`): 경량 Bitwarden 호환 비밀번호 관리 서버 (`8000`). 도메인 `vw.club012.com`.

---

## 실행 방법

각 스택 디렉토리에서 개별적으로 실행합니다. 비밀번호 및 주요 설정은 `.env` 파일 또는 환경변수를 통해 주입합니다.

```bash
cd <stack-dir>
docker compose up -d
```

### 루트 Dockhand 관리 UI 실행
```bash
docker compose up -d
```

### 공유 DB 및 Mattermost 실행 예시
```bash
# 1. 네트워크 생성 (최초 1회)
docker network create postgres

# 2. 공용 PostgreSQL 실행
cd database/postgres
docker compose up -d

# 3. Mattermost 실행 (공용 PostgreSQL 연동)
cd ../../channel/mattermost
docker compose up -d
```

---

## 참고 사항

- **도메인 네이밍 규약**: 외부 노출 서비스는 `*.club012.com` 형식으로 서브도메인이 할당됩니다 (예: `keycloak.club012.com`, `vault.club012.com`, `vw.club012.com`).
- **환경 변수 및 보안**: 각 디렉토리의 `.env` 파일 및 볼륨 데이터는 `.gitignore`에 등록되어 있습니다. 운영 환경 배포 전 반드시 기본 비밀번호(`POSTGRES_PASSWORD`, `MYSQL_ROOT_PASSWORD`, `KEYCLOAK_ADMIN_PASSWORD` 등)를 안전한 값으로 변경하세요.
