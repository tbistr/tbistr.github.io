FROM mcr.microsoft.com/vscode/devcontainers/base:debian-11

# install fish terminal
RUN apt-get update && apt-get install -y \
    fish \
    && rm -rf /var/lib/apt/lists/*

# install golang
ARG GO_VERSION=1.20.1
RUN set -x \
    && curl -fsSL "https://golang.org/dl/go${GO_VERSION}.linux-amd64.tar.gz" | tar -xz -C /usr/local
ENV PATH $PATH:/usr/local/go/bin
ENV PATH $PATH:/home/vscode/go/bin

ENV HUGO_VER=0.111.2
USER vscode
RUN CGO_ENABLED=1 go install --tags extended github.com/gohugoio/hugo@v${HUGO_VER}

USER root
RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive \
    apt-get install -y \
    nodejs npm && \
    npm install -g n && \
    n stable && \
    apt-get purge -y nodejs npm && \
    rm -rf /var/lib/apt/lists/*

RUN npm install -g \
    textlint \
    textlint-rule-preset-ja-spacing \
    textlint-rule-preset-ja-technical-writing \
    textlint-rule-spellcheck-tech-word
