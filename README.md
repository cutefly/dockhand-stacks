# dockhand-stacks

`club012.com` 인프라를 구성하는 Docker Compose 스택 모음. [Dockhand](https://github.com/fnsys/dockhand)로 컨테이너를 관리하며, 각 서비스는 독립된 디렉토리와 `compose.yaml`로 분리되어 있다.

## 구조

```
.
├── compose.yaml        # Dockhand 관리 UI (3000)
├── auth/                인증
│   ├── authentik/       (9000/9443) + postgres
│   ├── keycloak/        (8080) + postgres, OIDC 프로바이더
│   └── openldap/        (389/636) + phpldapadmin (8081)
├── cms/                  협업 / 스토리지
│   ├── docmost/          (3000) + postgres, redis
│   ├── redmine/          (3000) + mysql
│   └── rustfs/           (9000/9001) S3 호환 스토리지, KMS/OIDC 연동
├── database/             공용 DB
│   ├── mysql/            (3306)
│   ├── postgres/         (5432)
│   └── redis/            (6379)
├── devops/                CI/CD & 레지스트리
│   ├── gitlab/           (80/443), LDAP + Keycloak OIDC 연동
│   ├── harbor/           (80) 컨테이너 레지스트리, harbor.yml로 설정
│   ├── jenkins/          (8080/50000) + ssh-agent 2개
│   └── nexus/            (8081-8083) 아티팩트 저장소
├── kms/
│   └── vault/            (미구성, 빈 디렉토리)
└── secrets/               비밀 관리
    ├── vault/            (8200) HashiCorp Vault
    └── vaultwarden/      (8000) Bitwarden 호환 서버
```

## 인증/연동 구성

- **Keycloak**(`devops`용 realm)이 GitLab, RustFS의 OIDC 프로바이더 역할을 한다.
- **OpenLDAP**은 GitLab의 LDAP 로그인 소스로 연동되어 있다 (`base: ou=users,dc=club012,dc=com`).
- **RustFS**는 Vault Transit을 KMS 백엔드로 사용해 저장 데이터를 암호화할 수 있다 (`RUSTFS_KMS_ENABLE`).

## 실행

각 스택 디렉토리에서 개별적으로 실행한다. 대부분 `.env`(또는 쉘 환경변수)로 비밀번호·시크릿을 주입해야 하므로, 실행 전 필요한 값을 확인한다.

```bash
cd <stack-dir>
docker compose up -d
```

예: 루트의 Dockhand UI만 띄우려면

```bash
docker compose up -d
```

## 참고

- 서비스 도메인은 `*.club012.com` 형태로 통일되어 있다 (예: `gitlab.club012.com`, `keycloak.club012.com`, `rustfs.club012.com`, `vw.club012.com`).
- `harbor/harbor.yml`은 `harbor_admin_password`, DB 비밀번호가 평문으로 들어있으니 운영 반영 전 반드시 교체한다.
