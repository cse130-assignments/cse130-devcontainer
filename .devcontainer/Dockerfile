FROM debian:bullseye-slim AS haskell-env

ENV LANG C.UTF-8

# [Option] Install zsh
ARG INSTALL_ZSH="true"
# [Option] Upgrade OS packages to their latest versions
ARG UPGRADE_PACKAGES="false"

# Install needed packages and setup non-root user.
# Use a separate RUN statement to add your own dependencies
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID
COPY .devcontainer/library-scripts/*.sh /tmp/.devcontainer/library-scripts/
RUN apt-get update \
    && export DEBIAN_FRONTEND=noninteractive \
    && /bin/bash /tmp/.devcontainer/library-scripts/common-debian.sh "${INSTALL_ZSH}" "${USERNAME}" "${USER_UID}" "${USER_GID}" "${UPGRADE_PACKAGES}" "true" "true"\
    && rm -rf /tmp/.devcontainer/library-scripts \
    && apt-get install -y --no-install-recommends \
        dpkg-dev \
        gcc \
        g++ \
        libc6-dev \
        libffi-dev \
        libgmp-dev \
        libnuma-dev \
        libtinfo-dev \
        zlib1g-dev \
    && apt-get autoremove -y && apt-get clean -y && rm -rf /var/lib/apt/lists/*

# Install latest GHCup in the non-root user home
USER $USERNAME

RUN mkdir -p "$HOME/.ghcup/bin" \
    && curl -LJ "https://downloads.haskell.org/~ghcup/x86_64-linux-ghcup" -o "$HOME/.ghcup/bin/ghcup" \
    && chmod +x "$HOME/.ghcup/bin/ghcup"
ENV PATH="/home/$USERNAME/.cabal/bin:/home/$USERNAME/.ghcup/bin:$PATH"

# [Choice] GHC version: recommended, latest, 9.2, 9.0, 8.10, 8.8, 8.6
ARG GHC_VERSION=9.4.8

# Use GHCup to install versions of main utilities
# If you prefer to let the Haskell extension install everything on demand,
# comment out the lines below. In that case, you may need to manually run "cabal update""."
RUN ghcup install ghc "${GHC_VERSION}" --set \
    && ghcup install cabal recommended --set \
    && ghcup install stack recommended --set \
    && ghcup install hls recommended --set \
    && cabal update

RUN stack config set system-ghc --global true
# for sp25: explicitly added hie-bios-0.14.0 since lts-21.25 has an older/incompatible version.
RUN stack install --resolver=lts-21.25 hie-bios-0.14.0 haskell-dap ghci-dap haskell-debug-adapter 

# Install Elsa in separate container, so that no artifacts remain
FROM haskell-env AS elsa-build
WORKDIR workdir
RUN git clone https://github.com/ucsd-progsys/elsa.git elsa-repo
RUN cd elsa-repo && stack install --local-bin-path /workdir

# Copy *just* the elsa binary into our final container
FROM haskell-env AS final-img
COPY --from=elsa-build /workdir/elsa /usr/bin/elsa
