ARG IDF_VERSION=release-v5.3
FROM espressif/idf:${IDF_VERSION}

# Tools + Node (for Claude Code) + editor stack (Neovim + clangd LSP)
RUN apt-get update && apt-get install -y --no-install-recommends \
      zsh git curl ca-certificates ripgrep fzf gnupg \
      neovim clangd \
   && curl -fsSL https://deb.nodesource.com/setup_22.x | bash - \
   && apt-get install -y --no-install-recommends nodejs \
   && rm -rf /var/lib/apt/lists/*

# Claude Code CLI (uses mounted ~/.claude for credentials/session)
RUN npm install -g @anthropic-ai/claude-code

ARG USERNAME=dev
ARG USER_UID=2000
ARG USER_GID=2000

RUN groupadd --gid $USER_GID $USERNAME \
  && useradd  --uid $USER_UID --gid $USER_GID -m -s /usr/bin/zsh $USERNAME\
  && mkdir -p /workspaces && chown $USERNAME:$USERNAME /workspaces

USER $USERNAME
ENV RUNNING_IN_DEVCONTAINER=1
