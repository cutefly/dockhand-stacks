# Gitlab

## 배포 준비

```
git init dockhand-stacks
cd dockhand-stacks
git remote add -f origin https://github.com/cutefly/dockhand-stacks.git
echo "/devops/gitlab" > .git/sparse-checkout
git pull origin main
```
