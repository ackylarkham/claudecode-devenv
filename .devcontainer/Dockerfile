# ============================================================
# Claude Code 開発コンテナー
#
# ベースイメージ: node:24-bookworm-slim
#   - Node.js 24 "Krypton" = 現在の Active LTS（2028年4月まで保守）
#   - Debian 12 (Bookworm) slim = 最新安定版・最小フットプリント
#   - node:22-bookworm-slim より脆弱性が少なく、LTS として推奨
#
# 参考: https://nodejs.org/en/about/previous-releases
# ============================================================
#FROM node:24-bookworm-slim
FROM node:lts-bookworm-slim

# ------------------------------------------------------------
# タイムゾーン設定（東京）
# ------------------------------------------------------------
ENV TZ=Asia/Tokyo
RUN apt-get update && apt-get install -y --no-install-recommends \
    tzdata \
    && ln -snf /usr/share/zoneinfo/$TZ /etc/localtime \
    && echo $TZ > /etc/timezone \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# ------------------------------------------------------------
# 日本語ロケール設定
# ------------------------------------------------------------
RUN apt-get update && apt-get install -y --no-install-recommends \
    locales \
    && sed -i 's/# ja_JP.UTF-8 UTF-8/ja_JP.UTF-8 UTF-8/' /etc/locale.gen \
    && locale-gen ja_JP.UTF-8 \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

ENV LANG=ja_JP.UTF-8
ENV LANGUAGE=ja_JP:ja
ENV LC_ALL=ja_JP.UTF-8

# ------------------------------------------------------------
# 日本語フォント
# ------------------------------------------------------------
RUN apt-get update && apt-get install -y --no-install-recommends \
    fonts-noto-cjk \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# ------------------------------------------------------------
# 必要なシステムパッケージのインストール
# ------------------------------------------------------------
RUN apt-get update && apt-get install -y --no-install-recommends \
    # 要件指定パッケージ
    sudo \
    iputils-ping \
    iproute2 \
    openssh-client \
    # 基本ツール
    git \
    curl \
    wget \
    vim \
    bash \
    bash-completion \
    ca-certificates \
    procps \
    unzip \
    # ビルドツール（Claude Code / npm パッケージのネイティブモジュール用）
    build-essential \
    python3 \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# ------------------------------------------------------------
# IBM Semeru Java 21 JDK と graphviz のインストール
# Adoptium リポジトリには ibm-semeru パッケージが含まれないため
# GitHub リリースから直接ダウンロードする
# ------------------------------------------------------------
RUN apt-get update && apt-get install -y --no-install-recommends \
    graphviz \
    && apt-get clean && rm -rf /var/lib/apt/lists/* \
    && ARCH=$(dpkg --print-architecture) \
    && case ${ARCH} in \
       amd64)  IBM_ARCH="x64"     ;; \
       arm64)  IBM_ARCH="aarch64" ;; \
       *)      echo "Unsupported architecture: ${ARCH}" && exit 1 ;; \
    esac \
    && IBM_TAG="jdk-21.0.10%2B7.1_openj9-0.57.0" \
    && IBM_FILE="ibm-semeru-open-jdk_${IBM_ARCH}_linux_21.0.10.1.tar.gz" \
    && wget -qO /tmp/ibm-semeru.tar.gz \
       "https://github.com/ibmruntimes/semeru21-binaries/releases/download/${IBM_TAG}/${IBM_FILE}" \
    && mkdir -p /usr/lib/jvm \
    && tar -xzf /tmp/ibm-semeru.tar.gz -C /usr/lib/jvm \
    && rm /tmp/ibm-semeru.tar.gz \
    && JDK_DIR=$(ls /usr/lib/jvm | grep "^jdk-21") \
    && ln -s /usr/lib/jvm/${JDK_DIR} /usr/lib/jvm/ibm-semeru-open-21-jdk

ENV JAVA_HOME=/usr/lib/jvm/ibm-semeru-open-21-jdk
ENV PATH=${JAVA_HOME}/bin:${PATH}

# ------------------------------------------------------------
# 一般ユーザー「user」の作成
# ------------------------------------------------------------
ARG USERNAME=user
ARG GROUPNAME=user
ARG UID=2000
ARG GID=2000
ARG PASSWORD=user
RUN groupadd -g $GID $GROUPNAME && \
    useradd -m -s /bin/bash -u $UID -g $GID -G sudo $USERNAME && \
    echo $USERNAME:$PASSWORD | chpasswd && \
    echo "$USERNAME   ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers
USER $USERNAME

# ------------------------------------------------------------
# Claude Code のグローバルインストール
# ------------------------------------------------------------
RUN sudo npm install -g @anthropic-ai/claude-code

# ------------------------------------------------------------
# 作業ユーザーと作業ディレクトリの設定
# ------------------------------------------------------------
USER $USERNAME
# devcontainer 起動時にワークスペースへ自動 cd（ハードコードなし・動的検出）
RUN { \
    echo ''; \
    echo '# devcontainer: ワークスペースディレクトリへ自動移動'; \
    echo 'if [ "${REMOTE_CONTAINERS:-}" = "true" ] && [ "$(pwd)" = "$HOME" ]; then'; \
    echo '    _ws=$(find "$HOME" -maxdepth 1 -mindepth 1 -type d ! -name ".*" 2>/dev/null | sort | head -1)'; \
    echo '    [ -n "$_ws" ] && cd "$_ws"'; \
    echo '    unset _ws'; \
    echo 'fi'; \
    } >> /home/$USERNAME/.bashrc
WORKDIR /home/$USERNAME
