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
