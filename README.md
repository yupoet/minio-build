# minio-build（MinIO 社区版自维护分支）

上游 [minio/minio](https://github.com/minio/minio) 社区版于 **2026-04-25 归档（只读）**，
官方二进制分发渠道（`dl.min.io` 含 archive 与 latest 端点）随后全线 410，Docker Hub
镜像 tag 删除、quay.io 拉取 401。本仓库是这个最后版本的**自维护分发点**：

- **基线**：`RELEASE.2025-10-15T17-29-55Z`（官方最后一个 release，含 STS 权限提升
  CVE 修复），tag 已随仓库保留，历史完整可考古；上游原 README 见 `README.upstream.md`。
- **增量**：`build/` 目录——可复现的 Docker 镜像构建（本仓库唯一长期维护的部分；
  对上游源码的任何改动也会以提交形式出现在 main 分支，与 tag 可 diff）。
- **许可证**：AGPLv3，随上游 LICENSE 保留；本仓库含完整源码，分发合规。

## 构建（约 3 分钟）

```bash
git clone https://github.com/yupoet/minio-build.git && cd minio-build
docker build -f build/Dockerfile -t aide-minio:RELEASE.2025-10-15T17-29-55Z .
```

- 构建层 `golang:1.24-alpine`（实测解析到 go1.24.13，新于官方 09-07 版所用工具链，
  附带更新的 Go stdlib 安全修复）；运行层 `alpine:3.20`（官方 UBI registry 部分网络不可达）。
- `GOPROXY` 默认 `goproxy.cn,direct`（proxy.golang.org 在部分网络不稳，可按需覆盖）。
- 版本注入对齐官方 Makefile 的 ldflags 口径（`cmd.Version`/`cmd.ReleaseTag`）。

## 冒烟（换新版本/改代码后必做）

```bash
docker run --rm aide-minio:RELEASE.2025-10-15T17-29-55Z --version   # 版本号应一致
docker run -d --name minio-test -p 127.0.0.1:9900:9000 \
  -e MINIO_ROOT_USER=smoketest -e MINIO_ROOT_PASSWORD=smoketest123 \
  aide-minio:RELEASE.2025-10-15T17-29-55Z server /data
curl -sf http://127.0.0.1:9900/minio/health/live -o /dev/null        # 200
curl -sf http://127.0.0.1:9900/minio/health/ready -o /dev/null       # 200
docker rm -f minio-test
```

2026-09-26 首次构建实测全过（版本注入正确、双健康端点 200、镜像 42MB）。

## 维护策略

- 上游不会再有官方修复；Go 依赖链 CVE 的实际缓解是**定期用新 Go 工具链重建**
  （改 `golang:1.24-alpine` 为更新 tag 或直接重建即可）。
- 重建后跑冒烟；生产环境（Aide-Captain 的 WeKnora 栈）切换方式见其仓库
  `docker/weknora/minio-selfbuild/README.md`（compose 改 image 引用，数据卷同版本线兼容）。
- 长期方向是迁移 `STORAGE_TYPE=local` 消灭对象存储依赖（见 Aide-Captain runbook §15.4）；
  在那之前本仓库是唯一分发点，**构建产物需另行 `docker save` 归档**（本仓库管源码不管镜像）。
