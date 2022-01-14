FROM mcr.microsoft.com/vscode/devcontainers/go

RUN wget https://github.com/gohugoio/hugo/releases/download/v0.92.0/hugo_0.92.0_Linux-64bit.tar.gz \
    && tar -zxvf hugo_0.92.0_Linux-64bit.tar.gz -C /usr/bin \
    && rm -r hugo_0.92.0_Linux-64bit.tar.gz

RUN apt-get update && apt-get install -y \
    fish \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*
