# Jenkins

## 배포 준비

```
git init dockhand-stacks
cd dockhand-stacks
git branch -m master main
git config core.sparseCheckout true
git remote add -f origin https://github.com/cutefly/dockhand-stacks.git
echo "/devops/jenkins" > .git/info/sparse-checkout
git pull origin main
```
## Single sign on with keycloak

> https://muhammad-irfan-tahir.medium.com/part-1-keycloak-sso-with-jenkins-oidc-authentication-and-2fa-enforcement-b94096a82387

```
Keycloak Authentication Plugin 대신 OpenId Connect Authentication Plugin 사용
groups 매핑 이슈가 있음
```
