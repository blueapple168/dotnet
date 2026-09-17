## 概述

本目录用于构建基于银河麒麟 KylinOS V11 以及统信 UOS Server 20‑1070a 的`.NET runtime‑deps`基础容器镜像。

> 
> ⚠️ **镜像版本说明**
> 
> 
> - 银河麒麟：`cr.kylinos.cn/kylin/kylin-server-minimal:v11‑2503`（**此分支使用旧版 minimal 镜像，内置 microdnf 包管理器**；注意：后续发布的 2503 新版 minimal 镜像已经移除 microdnf，请勿混用基础镜像）
> - 统信 UOS：`uos-server-minimal:v20‑1070a`

## 目录结构

```
dotnet
├── kylinos
│   └── dotnet
│       └── runtime-deps     # KylinOS V11 .NET runtime‑deps Dockerfile & build脚本
└── uos
    └── dotnet
        └── runtime-deps     # UOS Server V20‑1070a .NET runtime‑deps Dockerfile & build脚本
```

## 镜像元信息规范

> 
> Dockerfile 内采用**OCI 标准 Annotation 标签（org.opencontainers.image.*）**，废弃旧的`maintainer`标签；兼容 Harbor、[cr.kylinos.cn](https://cr.kylinos.cn)镜像仓库与 Trivy 漏洞扫描识别。

**KylinOS LABEL 示例片段**

```
LABEL org.opencontainers.image.authors="blueapple" \
      org.opencontainers.image.version="1.0" \
      org.opencontainers.image.description="Dotnet runtime-deps" \
      org.opencontainers.image.base.name="cr.kylinos.cn/kylin/kylin-server-minimal" \
      org.opencontainers.image.base.version="${BASE_IMAGE_TAG}"
```

## 构建说明

### KylinOS‑V11（kylin‑server‑minimal:v11‑2503，内置 microdnf）

> 
> ✅本基础镜像自带`microdnf`，容器内部可直接执行安装 rpm 依赖包

```
cd dotnet/kylinos/dotnet/runtime-deps
# 示例构建命令
docker build \
  --build-arg BASE_IMAGE_TAG=v11-2503 \
  -t kylin-dotnet-runtime-deps:1.0 .
```

Dockerfile 内安装依赖写法（使用 microdnf）

```
RUN microdnf install -y xxx && microdnf clean all
```

> 
> ⚠️重要风险提示：
> `cr.kylinos.cn/kylin/kylin-server-minimal:v11‑2503`存在两个不同的镜像版本；**本项目锁定使用带有 microdnf 的旧版本 minimal 镜像。如拉取到新版无包管理器镜像会直接导致构建失败。升级基础镜像前务必确认镜像内部是否保留 microdnf。**

### UOS Server‑20‑1070a

```
cd dotnet/uos/dotnet/runtime-deps
docker build -t uos-dotnet-runtime-deps:1.0 .
```

## 使用方式

下游业务镜像基于本 runtime‑deps 镜像做多阶段构建，直接拷贝发布后的 dotnet 应用，不再重复安装系统依赖。

```
#下游业务镜像示例
FROM kylin-dotnet-runtime-deps:1.0
WORKDIR /app
COPY ./publish .
ENTRYPOINT ["./MyApp"]
```

## 镜像维护注意事项

1. **基础镜像锁定：KylinOS v11‑2503 必须保留旧版 minimal（带 microdnf），后续版本升级需要评估包管理器变更风险；**
2. 标签统一使用 OCI `org.opencontainers.image.*`注解，禁止继续使用废弃`MAINTAINER`指令；
3. 构建完成后建议执行 Trivy 漏洞扫描；
4. 镜像推送至内部容器仓库时，保留 OCI 元数据标签，便于仓库展示与资产治理；
5. UOS 1070a 版本需要跟踪统信官方镜像发布基线。

## 版本基线清单

表格

| OS | 基础镜像 | 包管理器 |
| --- | --- | --- |
| KylinOS V11‑2503 | [cr.kylinos.cn/kylin/kylin-server-minimal:v11](https://cr.kylinos.cn/kylin/kylin-server-minimal:v11‑2503) (旧版) | microdnf |
| UOS Server V20‑1070a | [uos-server-20-1070a:latest](registry.uniontech.com/uos-server-base/uos-server-20-1070a:latest) | yum |
