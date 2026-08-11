# RustFS high-performance, distributed object storage system

https://github.com/rustfs/rustfs

## Docker quick start

> The RustFS container runs as a non-root user rustfs (UID/GID 10001:10001). If you bind-mount host directories with Docker or Compose, every mounted path must be writable by that user, otherwise startup may fail with permission denied errors. This applies to data directories, log directories, and TLS certificate directories when RUSTFS_TLS_PATH is enabled.

```
# Create data and logs directories
mkdir -p data logs

# Change the owner of these directories
chown -R 10001:10001 data logs

# Using latest version
docker run -d -p 9000:9000 -p 9001:9001 -v $(pwd)/data:/data -v $(pwd)/logs:/logs rustfs/rustfs:latest

# Using specific version
docker run -d -p 9000:9000 -p 9001:9001 -v $(pwd)/data:/data -v $(pwd)/logs:/logs rustfs/rustfs:1.0.0-rc.1
```

## 배포 준비

```
git init dockhand-stacks
cd dockhand-stacks
git branch -m master main
git config core.sparseCheckout true
git remote add -f origin https://github.com/cutefly/dockhand-stacks.git
echo "cms/rustfs" > .git/info/sparse-checkout
git pull origin main
```