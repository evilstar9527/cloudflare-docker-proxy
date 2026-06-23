# cloudflare-docker-proxy

> ### ⚠️ **Important Notice**
> <span style="color:#d73a49;font-weight:bold">Docker Hub is rate-limiting Cloudflare Worker IPs, causing frequent <code>429</code> errors.</span>  
> <span style="color:#d73a49;font-weight:bold">This project is currently NOT recommended for production use.</span>


Due to the current instability, this project is not recommended for production use.
We will provide updates as soon as more information becomes available.


![deploy](https://github.com/ciiiii/cloudflare-docker-proxy/actions/workflows/deploy.yaml/badge.svg)

[![Deploy to Cloudflare Workers](https://deploy.workers.cloudflare.com/button)](https://deploy.workers.cloudflare.com/?url=https://github.com/ciiiii/cloudflare-docker-proxy)

> If you're looking for proxy for helm, maybe you can try [cloudflare-helm-proxy](https://github.com/ciiiii/cloudflare-helm-proxy).

## Deploy

1. click the "Deploy With Workers" button
2. follow the instructions to fork and deploy
3. update routes as you requirement

[![Deploy to Cloudflare Workers](https://deploy.workers.cloudflare.com/button)](https://deploy.workers.cloudflare.com/?url=https://github.com/ciiiii/cloudflare-docker-proxy)

## Routes configuration tutorial

1. use cloudflare worker host: only support proxy one registry
   ```javascript
   const routes = {
     "${workername}.${username}.workers.dev/": "https://registry-1.docker.io",
   };
   ```
2. use custom domain: support proxy multiple registries route by host
   - host your domain DNS on cloudflare
   - add `A` record of xxx.example.com to `192.0.2.1`
   - deploy this project to cloudflare workers
   - add `xxx.example.com/*` to HTTP routes of workers
   - add more records and modify the config as you need
   ```javascript
   const routes = {
     "docker.libcuda.so": "https://registry-1.docker.io",
     "quay.libcuda.so": "https://quay.io",
     "gcr.libcuda.so": "https://k8s.gcr.io",
     "k8s-gcr.libcuda.so": "https://k8s.gcr.io",
     "ghcr.libcuda.so": "https://ghcr.io",
   };
   ```

## 本部署使用说明（evilstarss.com）

本仓库已部署到 Cloudflare Workers，用于加速拉取以下公共镜像仓库：

| 子域名 | 上游 registry |
| --- | --- |
| `docker.evilstarss.com` | `registry-1.docker.io`（Docker Hub） |
| `quay.evilstarss.com` | `quay.io` |
| `gcr.evilstarss.com` | `gcr.io` |
| `k8s-gcr.evilstarss.com` | `k8s.gcr.io` |
| `k8s.evilstarss.com` | `registry.k8s.io` |
| `ghcr.evilstarss.com` | `ghcr.io` |

### 1. Docker Hub —— 可透明加速

把 [`examples/daemon.json`](examples/daemon.json) 写入 `/etc/docker/daemon.json` 后重启 Docker：

```bash
sudo cp examples/daemon.json /etc/docker/daemon.json
sudo systemctl restart docker
docker info | grep -A1 "Registry Mirrors"   # 应显示 docker.evilstarss.com
```

之后 `docker pull nginx` 这类 Docker Hub 镜像会自动走加速，**无需修改镜像名**。

### 2. 其它仓库 —— 需手动改镜像名前缀

`registry-mirrors` 只对 Docker Hub 生效；gcr / k8s / ghcr / quay 需把镜像名里的域名换成对应子域名：

```bash
docker pull gcr.evilstarss.com/google-containers/pause:3.9
docker pull k8s.evilstarss.com/pause:3.9            # registry.k8s.io
docker pull ghcr.evilstarss.com/<owner>/<image>:<tag>
docker pull quay.evilstarss.com/<org>/<image>:<tag>
```

### 修改配置 / 重新部署

编辑 `wrangler.toml` 的 `routes`，然后（已内置 `account_id` 与 `main`，一条命令即可）：

```bash
npx wrangler deploy --env production
```

> ⚠️ 注意事项
> - Docker Hub 会对 Cloudflare Workers 出口 IP 限流，`docker.*` 路径可能偶发 `429`；`gcr/k8s/ghcr/quay` 一般不受影响。
> - 免费版 Workers 限额约 10 万请求/天，自用足够，大流量拉取有被限/封风险。


