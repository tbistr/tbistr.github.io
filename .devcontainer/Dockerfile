FROM mcr.microsoft.com/devcontainers/javascript-node:20-buster

# install fish terminal
RUN apt-get update && apt-get install -y \
    fish \
    && rm -rf /var/lib/apt/lists/*

# install golang
# need for hugo modules
ARG GO_VERSION=1.20.1
RUN wget "https://golang.org/dl/go${GO_VERSION}.linux-amd64.tar.gz" && \
    tar -C /usr/local -xzf "go${GO_VERSION}.linux-amd64.tar.gz" && \
    rm "go${GO_VERSION}.linux-amd64.tar.gz"
ENV PATH $PATH:/usr/local/go/bin
ENV PATH $PATH:/home/vscode/go/bin

# install hugo
ARG HUGO_VER=0.121.1
RUN wget https://github.com/gohugoio/hugo/releases/download/v${HUGO_VER}/hugo_${HUGO_VER}_linux-amd64.tar.gz && \
    tar -C /usr/local/bin -xzf hugo_${HUGO_VER}_linux-amd64.tar.gz && \
    rm hugo_${HUGO_VER}_linux-amd64.tar.gz

RUN npm install -g \
    textlint \
    textlint-rule-preset-ja-spacing \
    textlint-rule-preset-ja-technical-writing \
    textlint-rule-spellcheck-tech-word
