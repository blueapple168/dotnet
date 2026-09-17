ARG BASE_IMAGE_TAG=uos-server-20-1070a:latest
FROM ghcr.io/blueapple168/${BASE_IMAGE_TAG} AS builder

LABEL maintainer="blueapple" \
      version="1.0" \
      description="Dotnet runtime-deps on UOS 1070a" \
      org.opencontainers.image.base.name="registry.uniontech.com/uos-server-base/uos-server-20-1070a:latest"

# 环境变量配置（保留原 .NET 核心环境变量）
ENV \
    # UID of the non-root user 'app'
    APP_UID=1654 \
    # Configure web servers to bind to port 8080 when present
    ASPNETCORE_HTTP_PORTS=8080 \
    # Enable detection of running in a container
    DOTNET_RUNNING_IN_CONTAINER=true

# 一次性执行所有安装、配置、清理操作，减少镜像层数（最佳实践）
RUN set -eux; \
    # 1. 更新统信仓库配置
    sed -i 's/\$StateMode/ufu/g' /etc/yum.repos.d/UnionTechOS.repo; \
    # 2. 生成缓存并安装依赖包
    yum makecache; \
    yum install -y --setopt=install_weak_deps=false --nogpgcheck --nodocs \
        # 基础工具
        tar \
        curl \
        ca-certificates \
        tzdata \
        # .NET 核心依赖（适配 UOS 系统包名）
        glibc \
        libgcc \
        libicu \
        openssl \
        libstdc++ \
        zlib; \
    # 3. 修复 Python setuptools 版本问题
    sed -i 's/Version: 39\.2\.0/Version: 2026.09.17/g' /usr/lib/python3.6/site-packages/setuptools-39.2.0.dist-info/METADATA 2>/dev/null || true; \
    # 4. 创建非 root 用户和用户组
    groupadd --gid=$APP_UID app; \
    useradd --no-log-init --uid=$APP_UID --gid=$APP_UID --create-home app; \
    # 5. 清理缓存和临时文件，减小镜像体积
    yum clean all; \
    rm -rf \
        /var/cache/yum/* \
        /var/tmp/* \
        /tmp/* \
        /var/lib/apt/lists/*
