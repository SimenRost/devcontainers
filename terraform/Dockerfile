# syntax=docker/dockerfile:1

FROM mcr.microsoft.com/devcontainers/base:bullseye@sha256:8f85d8372ecdf793545e222474cab6fe2803ce76a42668d9ed471f3da6663410 AS baseimage

RUN \
    usermod -s /bin/zsh vscode && \
    rm -rf /home/vscode/.oh-my-zsh

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
ENV JNV_VERSION=0.2.2

RUN \
    mkdir -p /tmp/jnv /out/usr/bin && \
    wget -O /tmp/jnv.tar.gz https://github.com/ynqa/jnv/releases/download/v$JNV_VERSION/jnv-x86_64-unknown-linux-gnu.tar.xz && \
    tar xf \
    /tmp/jnv.tar.gz -C \
    /tmp/jnv/ --strip-components=1 && \
    mv /tmp/jnv/jnv /out/usr/bin/jnv

FROM  busybox:1.36.1-glibc AS fzf

# renovate: datasource=github-releases depName=junegunn/fzf
ENV FZF_VERSION=0.52.0

RUN \
    mkdir -p /tmp/fzf /out/usr/bin && \
    wget -O /tmp/fzf.tar.gz https://github.com/junegunn/fzf/releases/download/$FZF_VERSION/fzf-$FZF_VERSION-linux_amd64.tar.gz && \
    tar xf \
    /tmp/fzf.tar.gz -C \
    /tmp/fzf/ --strip-components=1 && \
    mv /tmp/fzf/fzf /out/usr/bin/fzf

FROM  busybox:1.36.1-glibc AS pwsh

# renovate: datasource=github-releases depName=PowerShell/PowerShell
ENV PWSH_VERSION=7.4.2

RUN \
    mkdir -p /tmp/pwsh /out/pwsh && \
    wget -O /tmp/pwsh.tar.gz https://github.com/PowerShell/PowerShell/releases/download/v$PWSH_VERSION/powershell-$PWSH_VERSION-linux-x64.tar.gz  && \
    tar xf \
    /tmp/pwsh.tar.gz -C \
    /tmp/pwsh/ && \
    ls -la && \
    mv /tmp/pwsh /out

FROM scratch AS binlayer

COPY --from=hashicorp/terraform:1.8.3 /bin/terraform /out/usr/bin/terraform

COPY --from=ghcr.io/homebrew/core/terraform-docs:0.17.0 /terraform-docs/*/bin/* /out/usr/bin
COPY --from=ghcr.io/homebrew/core/terraform-docs:0.17.0 /terraform-docs/*/etc/* /out/etc
COPY --from=ghcr.io/homebrew/core/terraform-docs:0.17.0 /terraform-docs/*/share/* /out/usr/local/share/

COPY --from=ghcr.io/jqlang/jq:1.7.1 /jq /out/usr/bin/jq

COPY --from=bat /out /out
COPY --from=jnv /out /out
COPY --from=pwsh /out /out


FROM baseimage

COPY --from=binlayer --chmod=755 --chown=root:root /out /

RUN \
    sudo chown vscode:vscode -R /usr/local/share/zsh/site-functions && \
    sudo chmod 755 -R /usr/local/share/zsh/site-functions && \
    ln -s /pwsh/pwsh /usr/bin/pwsh
