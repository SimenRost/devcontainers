# syntax=docker/dockerfile:1

FROM mcr.microsoft.com/devcontainers/base:bookworm@sha256:da67c59f82e057e0ccb81ec7d13fb50464c26f477638018e6de1b2623a008a3a AS baseimage

RUN \
  rm -rf /home/vscode/.oh-my-zsh && \
  rm -rf /home/vscode/.zshrc && \
  apt update && \
  apt install -y --no-install-recommends \
  file \
  tmux \
  vim && \
  usermod -s /bin/zsh vscode && \
  rm -rf \
  /tmp/* \
  /var/lib/apt/lists/* \
  /var/tmp/*

FROM busybox:1.37.0-glibc AS bat

# renovate: datasource=github-releases depName=sharkdp/bat
ENV BAT_VERSION=0.25.0

RUN \
  mkdir -p /tmp/bat /out/usr/bin /out/usr/local/share/zsh/site-functions /out/etc/bash_completion.d && \
  wget -O /tmp/bat.tar.gz https://github.com/sharkdp/bat/releases/download/v$BAT_VERSION/bat-v$BAT_VERSION-x86_64-unknown-linux-gnu.tar.gz && \
  tar xf \
  /tmp/bat.tar.gz -C \
  /tmp/bat/ --strip-components=1 && \
  mv /tmp/bat/bat /out/usr/bin/bat && \
  mv /tmp/bat/autocomplete/bat.zsh /out/usr/local/share/zsh/site-functions/_bat && \
  mv /tmp/bat/autocomplete/bat.bash /out/etc/bash_completion.d/bat

FROM  busybox:1.37.0-glibc AS jnv

# renovate: datasource=github-releases depName=ynqa/jnv
ENV JNV_VERSION=0.6.0

RUN \
  mkdir -p /tmp/jnv /out/usr/bin && \
  wget -O /tmp/jnv.tar.gz https://github.com/ynqa/jnv/releases/download/v$JNV_VERSION/jnv-x86_64-unknown-linux-gnu.tar.xz && \
  tar xf \
  /tmp/jnv.tar.gz -C \
  /tmp/jnv/ --strip-components=1 && \
  mv /tmp/jnv/jnv /out/usr/bin/jnv

FROM  busybox:1.37.0-glibc AS fzf

# renovate: datasource=github-releases depName=junegunn/fzf
ENV FZF_VERSION=v0.63.0

RUN \
  mkdir -p /tmp/fzf /out/usr/bin && \
  wget -O /tmp/fzf.tar.gz https://github.com/junegunn/fzf/releases/download/$FZF_VERSION/fzf-${FZF_VERSION#v}-linux_amd64.tar.gz && \
  wget -O /out/usr/bin/fzf-tmux https://raw.githubusercontent.com/junegunn/fzf/$FZF_VERSION/bin/fzf-tmux && \
  tar xf \
  /tmp/fzf.tar.gz -C \
  /tmp/fzf/ && \
  mv /tmp/fzf/fzf /out/usr/bin/fzf

FROM  busybox:1.37.0-glibc AS eza

# renovate: datasource=github-releases depName=eza-community/eza
ENV EZA_VERSION=0.21.6

RUN \
  mkdir -p /tmp/eza /tmp/eza_comp /out/usr/local/share/zsh/site-functions /out/etc/bash_completion.d /out/usr/bin && \
  wget -O /tmp/eza.tar.gz https://github.com/eza-community/eza/releases/download/v$EZA_VERSION/eza_x86_64-unknown-linux-gnu.tar.gz  && \
  wget -O /tmp/eza_comp.tar.gz https://github.com/eza-community/eza/releases/download/v$EZA_VERSION/completions-$EZA_VERSION.tar.gz  && \
  tar xf \
  /tmp/eza.tar.gz -C \
  /tmp/eza/ --strip-components=1 && \
  tar xf \
  /tmp/eza_comp.tar.gz -C \
  /tmp/eza_comp/ --strip-components=1 && \
  mv /tmp/eza/eza /out/usr/bin/eza && \
  mv /tmp/eza_comp/target/completions-$EZA_VERSION/_eza /out/usr/local/share/zsh/site-functions/_eza && \
  mv /tmp/eza_comp/target/completions-$EZA_VERSION/eza /out/etc/bash_completion.d/eza

FROM  busybox:1.37.0-glibc AS omp

# renovate: datasource=github-releases depName=jandedobbeleer/oh-my-posh
ENV OMP_VERSION=26.13.0

RUN \
  mkdir -p /out/usr/bin && \
  wget -O /tmp/omp https://github.com/JanDeDobbeleer/oh-my-posh/releases/download/v$OMP_VERSION/posh-linux-amd64 && \
  mv /tmp/omp /out/usr/bin/oh-my-posh

FROM  busybox:1.37.0-glibc AS pwsh

# renovate: datasource=github-releases depName=PowerShell/PowerShell
ENV PWSH_VERSION=7.5.2

RUN \
  mkdir -p /tmp/pwsh /out/pwsh && \
  wget -O /tmp/pwsh.tar.gz https://github.com/PowerShell/PowerShell/releases/download/v$PWSH_VERSION/powershell-$PWSH_VERSION-linux-x64.tar.gz  && \
  tar xf \
  /tmp/pwsh.tar.gz -C \
  /tmp/pwsh/ && \
  ls -la && \
  mv /tmp/pwsh /out

FROM scratch AS binlayer

COPY --from=ghcr.io/roxedus/dotfiles:latest@sha256:30bf13e8c8f8becbe392a19806ec6458718b772451c96059fe3161cd9c6df03a /out /out/home/vscode

COPY --from=hashicorp/terraform:1.12.2 /bin/terraform /out/usr/bin/terraform

COPY --from=ghcr.io/homebrew/core/terraform-docs:0.20.0 /terraform-docs/*/bin/* /out/usr/bin
COPY --from=ghcr.io/homebrew/core/terraform-docs:0.20.0 /terraform-docs/*/etc/* /out/etc
COPY --from=ghcr.io/homebrew/core/terraform-docs:0.20.0 /terraform-docs/*/share/* /out/usr/local/share/

COPY --from=ghcr.io/jqlang/jq:1.8.0 /jq /out/usr/bin/jq

COPY --from=bat /out /out
COPY --from=eza /out /out
COPY --from=fzf /out /out
COPY --from=jnv /out /out
COPY --from=omp /out /out
COPY --from=pwsh /out /out


FROM baseimage

COPY --from=binlayer --chmod=755 --chown=root:root /out /

RUN \
  sudo chown vscode:vscode -R /usr/local/share/zsh/site-functions && \
  sudo chown vscode:vscode -R /home/vscode && \
  sudo chmod 755 -R /usr/local/share/zsh/site-functions && \
  rm /home/vscode/.gitconfig && \
  ln -s /pwsh/pwsh /usr/bin/pwsh
