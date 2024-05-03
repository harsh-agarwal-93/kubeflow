#
# NOTE: Use the Makefiles to build this image correctly.
#

ARG BASE_IMG=<codeserver>
FROM $BASE_IMG

ARG TARGETARCH

USER root

# args - software versions
ARG CODESERVER_PYTHON_VERSION=2023.18.0
ARG PIP_VERSION=23.3.2
ARG PYTHON_VERSION=3.11.8
# ARG POETRY_VERSION=1.7.1
# ARG PRE_COMMIT_VERSION=3.5.0

ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1
ENV DEBIAN_FRONTEND noninteractive

# install - usefull linux packages
RUN wget -O- https://apt.releases.hashicorp.com/gpg | gpg --dearmor | tee /usr/share/keyrings/hashicorp-archive-keyring.gpg \
 && echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | tee /etc/apt/sources.list.d/hashicorp.list \
 && apt-get -y update \
 && apt-get -y upgrade \
 && apt-get -y install --no-install-recommends \
 ssh postgresql-client git bash-completion terraform \
 gnupg unixodbc-dev openssl curl ccache \
 apt-transport-https ca-certificates software-properties-common \
 llvm clang xz-utils libxml2-dev libxmlsec1-dev \
 build-essential gdb lcov pkg-config \
 libbz2-dev libffi-dev libgdbm-dev libgdbm-compat-dev liblzma-dev \
 libncurses5-dev libreadline6-dev libsqlite3-dev libssl-dev \
 lzma lzma-dev tk-dev uuid-dev zlib1g-dev \
 && update-ca-certificates \
 && apt-get clean \
 && rm -rf /var/lib/apt/lists/*
COPY --chown=${NB_USER}:users terraform-plugins.tgz /tmp
RUN mkdir -p /usr/local/share/terraform/plugins \
 && tar xzf /tmp/terraform-plugins.tgz -C /usr/local/share/terraform/plugins \
 && chown -R ${NB_USER}:users /usr/local/share/terraform/plugins \
 && rm -f /tmp/terraform-plugins.tgz

# Build python with an optimized mode using clang
# Flags to set quiet some standard warnings which are specifically superfluous to CPython and ccache
ENV CFLAGS="-Wno-unused-value -Wno-empty-body -Qunused-arguments -Wno-parentheses-equality"

RUN mkdir /python \
 && curl -fsSL https://www.python.org/ftp/python/3.11.8/Python-3.11.8.tgz \
    | tar xz -C /python --strip-components 1 \
 && cd /python \
 && CC=clang ./configure --enable-optimizations --with-lto \
 && make -j \
 && make install \
 && rm -rf /python

# setup environment
RUN chown -R ${NB_USER}:users ${HOME}

USER $NB_UID
WORKDIR ${HOME}

# python optimization flags for performance mode
# increase python build times
# ENV PYTHON_CONFIGURE_OPTS='--enable-optimizations --with-lto'
# ENV PYTHON_CFLAGS='-march=native -mtune=native'

# install python version
ENV PATH=${HOME}/.local/bin:${PATH}
RUN echo 'alias python="python3"' >> ~/.bash_aliases \
 && echo 'alias pip="pip3"' >> ~/.bash_aliases \
 && pip3 install pre-commit poetry \
 && poetry config virtualenvs.in-project true \
 && pre-commit init-templatedir ~/.git-template \
 && git config --global init.templateDir ~/.git-template

# install - codeserver extensions
RUN code-server --install-extension "ms-python.python@${CODESERVER_PYTHON_VERSION}" --force \
   --install-extension "DavidAnson.vscode-markdownlint" \
   --install-extension "GrapeCity.gc-excelviewer" \
   --install-extension "HashiCorp.terraform" \
   --install-extension "bierner.markdown-mermaid" \
   --install-extension "bierner.markdown-preview-github-styles" \
   --install-extension "ckolkman.vscode-postgres" \
   --install-extension "eamodio.gitlens" \
   --install-extension "hediet.vscode-drawio" \
   --install-extension "kevinrose.vsc-python-indent" \
   --install-extension "mechatroner.rainbow-csv" \
   --install-extension "mitchdenny.ecdc" \
   --install-extension "ms-python.isort" \
   --install-extension "ms-toolsai.jupyter" \
   --install-extension "ms-vscode.Theme-MarkdownKit" \
   --install-extension "redhat.vscode-xml" \
   --install-extension "redhat.vscode-yaml" \
   --install-extension "shd101wyy.markdown-preview-enhanced" \
   --install-extension "streetsidesoftware.code-spell-checker" \
   --install-extension "AdamRaichu.pdf-viewer@1.1.1" \
   --install-extension "yzane.markdown-pdf" \
   --install-extension "yzhang.markdown-all-in-one" \
 && code-server --list-extensions --show-versions

# install AWS CLI
USER root
RUN curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "/tmp/awscliv2.zip" \
   && unzip -q /tmp/awscliv2.zip \
   && ./aws/install \
   && rm -f /tmp/awscliv2.zip \
   && rm -rf aws

# s6 - 01-copy-tmp-home
RUN mkdir -p /tmp_home \
 && cp -r ${HOME} /tmp_home \
 && chown -R ${NB_USER}:users /tmp_home
USER $NB_UID
