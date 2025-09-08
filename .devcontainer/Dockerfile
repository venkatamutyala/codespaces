# See here for image contents: https://github.com/microsoft/vscode-dev-containers/tree/v0.233.0/containers/ubuntu/.devcontainer/base.Dockerfile

# [Choice] Ubuntu version (use ubuntu-22.04 or ubuntu-18.04 on local arm64/Apple Silicon): ubuntu-22.04, ubuntu-20.04, ubuntu-18.04
#https://mcr.microsoft.com/v2/devcontainers/base/tags/list
#https://github.com/devcontainers/images/tree/main/src/base-ubuntu
FROM mcr.microsoft.com/vscode/devcontainers/base:2-ubuntu-22.04@sha256:e57fdff274ed0bac0776490fa2644c242491bb21d4df7ad0bb79b433eac5168d

RUN apt update \
    && apt install tmux dnsutils telnet iputils-ping jq certbot nethogs nload vim htop -y \
    && apt clean -y

# Install tools to /usr/local/bin
COPY tools/ /etc/tools/
RUN for file in /etc/tools/*; do \
        if [ -f "$file" ]; then \
            new_name=$(basename "$file" .sh); \
            cp "$file" "/usr/local/bin/$new_name"; \
            chmod +x "/usr/local/bin/$new_name"; \
            echo "Copied and made executable: $new_name"; \
        fi; \
    done && \
    rm -rf /etc/tools

# Install code tunnel so we can run outside of github codespaces easily
RUN wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > packages.microsoft.gpg \
    && sudo install -D -o root -g root -m 644 packages.microsoft.gpg /etc/apt/keyrings/packages.microsoft.gpg \
    && sudo sh -c 'echo "deb [arch=amd64,arm64,armhf signed-by=/etc/apt/keyrings/packages.microsoft.gpg] https://packages.microsoft.com/repos/code stable main" > /etc/apt/sources.list.d/vscode.list' \
    && rm -f packages.microsoft.gpg \
    && apt update \
    && apt install code wget gpg apt-transport-https -y \
    && apt clean -y \
    && curl -Lk 'https://code.visualstudio.com/sha/download?build=stable&os=cli-alpine-x64' --output vscode_cli.tar.gz \
    && tar -xf vscode_cli.tar.gz \
    && rm vscode_cli.tar.gz

#AWS Debian has a user of admin that uses the UID of 1000. THis is to try and fix that. We are also using UID of 1337 on the system/host level here: https://github.com/GlueOps/development-only-utilities/blob/main/tools/developer-setup/linux-setup.sh#L8
RUN usermod --uid 1337 vscode --shell /bin/zsh

ARG NONROOT_USER=vscode
RUN echo "#!/bin/sh\n\
    sudoIf() { if [ \"\$(id -u)\" -ne 0 ]; then sudo \"\$@\"; else \"\$@\"; fi }\n\
    SOCKET_GID=\$(stat -c '%g' /var/run/docker.sock) \n\
    if [ \"${SOCKET_GID}\" != '0' ]; then\n\
        if [ \"\$(cat /etc/group | grep :\${SOCKET_GID}:)\" = '' ]; then sudoIf groupadd --gid \${SOCKET_GID} docker-host; fi \n\
        if [ \"\$(id ${NONROOT_USER} | grep -E \"groups=.*(=|,)\${SOCKET_GID}\(\")\" = '' ]; then sudoIf usermod -aG \${SOCKET_GID} ${NONROOT_USER}; fi\n\
    fi\n\
    exec \"\$@\"" > /usr/local/share/docker-init.sh \
    && chmod +x /usr/local/share/docker-init.sh

# VS Code overrides ENTRYPOINT and CMD when executing `docker run` by default.
# Setting the ENTRYPOINT to docker-init.sh will configure non-root access to
# the Docker socket if "overrideCommand": false is set in devcontainer.json.
# The script will also execute CMD if you need to alter startup behaviors.

USER vscode

RUN code --install-extension MS-vsliveshare.vsliveshare --extensions-dir /home/vscode/.vscode-remote/extensions
RUN code --install-extension GitHub.codespaces --extensions-dir /home/vscode/.vscode-remote/extensions

RUN mkdir -p /home/vscode/.vscode-server
RUN ln -s /home/vscode/.vscode-remote/extensions /home/vscode/.vscode-server/extensions

RUN git config --global --add --bool push.autoSetupRemote true

# Add backup warning to .zshrc for vscode user
RUN echo '\n# === Backup & Git Reminder ===' | tee -a /home/vscode/.zshrc && \
    echo 'echo -e "\e[1;31m⚠️  WARNING: No backups are configured. You are responsible for any data loss.\e[0m"' | tee -a /home/vscode/.zshrc && \
    echo 'echo -e "\e[1;33m💡 Tip: Use '\''git commit'\'' and '\''git push'\'' regularly to avoid losing your work.\e[0m"' | tee -a /home/vscode/.zshrc && \
    echo '# =============================' | tee -a /home/vscode/.zshrc && \
    chown vscode:vscode /home/vscode/.zshrc

USER root
RUN chown -R vscode:vscode /home/vscode
ENTRYPOINT [ "/usr/local/share/docker-init.sh" ]
CMD [ "sleep", "infinity" ]
