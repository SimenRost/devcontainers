# syntax=docker/dockerfile:1

FROM mcr.microsoft.com/devcontainers/base:bullseye@sha256:7c391a762d4ec15a6c279189a03ed3582aa8dadda8300921562cc95ddf100816 AS baseimage

RUN \
    usermod -s /bin/zsh vscode

FROM busybox:1.36.1-glibc AS bat

# renovate: datasource=github-releases depName=sharkdp/bat
ENV BAT_VERSION=0.24.0

RUN \
    mkdir -p /tmp/bat /out/usr/bin /out/usr/local/share/zsh/site-functions /out/etc/bash_completion.d && \
    wget -O /tmp/bat.tar.gz https://github.com/sharkdp/bat/releases/download/v$BAT_VERSION/bat-v$BAT_VERSION-x86_64-unknown-linux-gnu.tar.gz && \
    tar xf \
    /tmp/bat.tar.gz -C \
    /tmp/bat/ --strip-components=1 && \
    mv /tmp/bat/bat /out/usr/bin/bat && \
    mv /tmp/bat/autocomplete/bat.zsh /out/usr/local/share/zsh/site-functions/_bat && \
    mv /tmp/bat/autocomplete/bat.bash /out/etc/bash_completion.d/bat

FROM  busybox:1.36.1-glibc AS jnv

# renovate: datasource=github-releases depName=ynqa/jnv
ENV JNV_VERSION=0.2.1

RUN \
    mkdir -p /tmp/jnv /out/usr/bin && \
    wget -O /tmp/jnv.tar.gz https://github.com/ynqa/jnv/releases/download/v$JNV_VERSION/jnv-x86_64-unknown-linux-gnu.tar.xz && \
    tar xf \
    /tmp/jnv.tar.gz -C \
    /tmp/jnv/ --strip-components=1 && \
    ls /tmp/ && \
    mv /tmp/jnv/jnv /out/usr/bin/jnv

FROM scratch AS binlayer

COPY --from=hashicorp/terraform:1.7.5 /bin/terraform /out/usr/bin/terraform

COPY --from=ghcr.io/homebrew/core/terraform-docs:0.17.0 /terraform-docs/*/bin/* /out/usr/bin
COPY --from=ghcr.io/homebrew/core/terraform-docs:0.17.0 /terraform-docs/*/etc/* /out/etc
COPY --from=ghcr.io/homebrew/core/terraform-docs:0.17.0 /terraform-docs/*/share/* /out/usr/local/share/

COPY --from=ghcr.io/jqlang/jq:1.7.1 /jq /out/usr/bin/jq

COPY --from=bat /out /out
COPY --from=jnv /out /out

FROM baseimage

COPY --from=binlayer --chmod=755 --chown=root:root /out /

RUN \
    sudo chown vscode:vscode -R /usr/local/share/zsh/site-functions && \
    sudo chmod 755 -R /usr/local/share/zsh/site-functions
