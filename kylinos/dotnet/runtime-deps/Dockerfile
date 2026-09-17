ARG BASE_IMAGE_TAG=v11-2503
FROM ghcr.io/blueapple168/kylin-server-minimal:${BASE_IMAGE_TAG}

LABEL org.opencontainers.image.authors="blueapple" \
      org.opencontainers.image.version="1.0" \
      org.opencontainers.image.description="Dotnet runtime-deps" \
      org.opencontainers.image.base.name="cr.kylinos.cn/kylin/kylin-server-minimal" \
      org.opencontainers.image.base.version="${BASE_IMAGE_TAG}"

ENV \
    APP_UID=1654 \
    ASPNETCORE_HTTP_PORTS=8080 \
    DOTNET_RUNNING_IN_CONTAINER=true

# 安装所有 .NET 必需依赖 + 用户命令 + 基础工具
RUN set -eux; \
    microdnf install -y \
        # 用户管理(useradd/groupadd)
        shadow \
        # 证书/时区
        ca-certificates \
        tzdata \
        # .NET 核心依赖库
        libicu \
        openssl \
        zlib \
        # 解决 "required file not found" 关键依赖
        glibc \
        libgcc \
        libstdc++ \
        # 工具
        vim \
        curl \
        tar \
    ; \
    microdnf clean all; \
    rm -rf /var/cache/yum/* /var/tmp/* /tmp/*

# 创建非root用户
RUN groupadd --gid=$APP_UID app \
    && useradd --no-log-init --uid=$APP_UID --gid=$APP_UID --create-home app
