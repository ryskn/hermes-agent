# syntax=docker/dockerfile:1
# 公式イメージに、claude-code-cli provider (upstream PR #98533) を当てたソースと Claude Code の CLI を足す。
# 公式イメージはソースを /opt/hermes に置いたまま .pth で参照するので、変更したファイルを上書きすれば反映される。
# ビルドの context は .github/workflows/opc-image.yml が作る (BASE_REF から変更したファイルだけを入れる)。
ARG BASE_IMAGE
FROM ${BASE_IMAGE}

ARG CLAUDE_CODE_VERSION
USER root

# claude-code-cli provider は Hermes から `claude` を子プロセスとして起動する。
# npm は allowScripts に無いパッケージの postinstall を止めるので、ネイティブバイナリを置く install.cjs を直接実行する。
RUN npm install --prefix /opt/claude-code --omit=dev --no-fund --no-audit \
      "@anthropic-ai/claude-code@${CLAUDE_CODE_VERSION}" && \
    cd /opt/claude-code/node_modules/@anthropic-ai/claude-code && node install.cjs && cd / && \
    ln -sf /opt/claude-code/node_modules/.bin/claude /usr/local/bin/claude && \
    claude --version

COPY --link --chmod=a+rX,go-w . /opt/hermes/
RUN /opt/hermes/.venv/bin/python -m compileall -q /opt/hermes/agent /opt/hermes/hermes_cli \
      /opt/hermes/plugins/model-providers/claude-code /opt/hermes/run_agent.py && \
    /opt/hermes/.venv/bin/python -c "import agent.claude_code_runtime, agent.transports.claude_code_session"
