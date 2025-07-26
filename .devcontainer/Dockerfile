# syntax=docker/dockerfile:1.8
# check=error=true

## Multi-stage build!
# Pull latest prebuilt Echidna binary.
# TODO: "Ensure the base image uses a non latest version tag"
# FROM --platform=linux/amd64 ghcr.io/crytic/echidna/echidna:latest AS echidna
FROM ghcr.io/crytic/echidna/echidna:latest AS echidna

FROM ghcr.io/a16z/halmos:latest AS halmos

# Grab at least python 3.12
FROM python:3.13-slim as python-base

# Base ubuntu build (latest).
# FROM mcr.microsoft.com/vscode/devcontainers/base:ubuntu // I think this doesn't have new versions of GLIBC
FROM mcr.microsoft.com/devcontainers/base:ubuntu-24.04

# Switch to root (the default might be root anyway)
USER root

COPY --from=python-base /usr/local /usr/local

# Super basic stuff to get everything started
RUN apt-get update -y && apt-get install -y \
    fish zsh python3-dev libpython3-dev build-essential vim curl git sudo pkg-config \
    --no-install-recommends

# # The base container usually has a “vscode” user. If not, create one here.
# RUN echo "vscode ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

# Dropping privileges
USER vscode

# NOTE: If you replace fish with bash/zsh shell then you need to change variables in command. Like in fish shell, Variables cannot be bracketed. For example, in fish, it will be {$HOME} instead of ${HOME}.
# Set the default shell to fish
ENV SHELL=/usr/bin/fish

# Running everything under fish
SHELL ["/usr/bin/fish", "-c"]

# Install starship theme
RUN curl -sS https://starship.rs/install.sh | sh -s -- --yes

# Copy the starship config file
RUN mkdir -p $HOME/.config
COPY ./starship.toml $HOME/.config/starship.toml

# Config starship for fish ( https://starship.rs/#fish )
RUN echo 'starship init fish | source' >> $HOME/.config/fish/config.fish

# Config starship for zsh ( https://starship.rs/#zsh )
RUN echo 'eval "$(starship init zsh)"' >> $HOME/.zshrc

# Config starship for bash ( https://starship.rs/#bash )
RUN echo 'eval "$(starship init bash)"' >> $HOME/.bashrc

# The below 2 lines are required for zsh to load completions
USER root
RUN mkdir -p /usr/share/zsh/site-functions
USER vscode
# # Refer ( https://www.reddit.com/r/zsh/comments/1fx4ftd/comment/lqjp6r3 )
RUN echo 'fpath=($fpath /usr/share/zsh/site-functions/)' >> $HOME/.zshrc
RUN echo 'autoload -Uz compinit && compinit' >> $HOME/.zshrc

# All the auto completion scripts for bash should be in ~/.config/bash/completions/ directory ( I don't know if there's already an ideal way of doing it )
RUN mkdir -p ~/.config/bash/completions && echo -e "\nfor f in ~/.config/bash/completions/*; do source \$f; done" >> ~/.bashrc

# # Install clang
# RUN sudo apt-get install -y clang

# # Install Make
# RUN sudo apt-get install -y make

# Install asdf and some plugins
# Set asdf manager version
ENV ASDF_VERSION=v0.15.0
RUN git clone https://github.com/asdf-vm/asdf.git $HOME/.asdf --branch {$ASDF_VERSION} \
    # Refer https://asdf-vm.com/guide/getting-started.html#_3-install-asdf

    # Config asdf for fish
    && echo 'source $HOME/.asdf/asdf.fish' >> $HOME/.config/fish/config.fish \
    && mkdir -p $HOME/.config/fish/completions; and ln -s $HOME/.asdf/completions/asdf.fish $HOME/.config/fish/completions \

    # Config asdf for zsh
    && echo '. $HOME/.asdf/asdf.sh' >> $HOME/.zshrc \
    && echo '. $HOME/.asdf/completions/asdf.bash' >> $HOME/.zshrc \

    # # If shell is bash
    # && . $HOME/.asdf/asdf.sh \

    # If shell is fish
    && source $HOME/.asdf/asdf.fish

# Install asdf plugins
RUN asdf plugin add scarb \
    && asdf install scarb latest \
    # && asdf install scarb 0.7.0 \
    # && asdf install scarb 2.6.5 \
    # && asdf install scarb 2.8.3 \
    && asdf plugin add starknet-foundry \
    && asdf install starknet-foundry latest \
    # && asdf install starknet-foundry 0.31.0 \
    && asdf plugin add golang \
    && asdf install golang latest

# Set the plugins to the latest version
# # asdf global <name> <version>            Set the package global version
# # asdf global <name> latest[:<version>]   Set the package global version to the latest provided version
RUN asdf global scarb latest
RUN asdf global starknet-foundry latest
RUN asdf global golang latest

# # Install rust version 1.80.0
# RUN curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- --default-toolchain=1.80.0 -y \
# Install rust latest version
RUN curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y \

    # To get started you may need to restart your current shell.
    # This would reload your PATH environment variable to include
    # Cargo's bin directory ($HOME/.cargo/bin).

    # # For sh/bash/zsh/ash/dash/pdksh shell
    # . $HOME/.cargo/env

    # For fish shell
    && source $HOME/.cargo/env.fish

#######################################
# REFER: https://nodejs.org/en/download

# # Download and install fnm:
# RUN curl -fsSL https://fnm.vercel.app/install | bash

# # Not using curl as it's hosted in vercel. Using cargo ( refer https://github.com/Schniz/fnm?tab=readme-ov-file#using-cargo-linuxmacoswindows )
RUN cargo install fnm

# # As installed with cargo, Shell setup is required ( refer https://github.com/Schniz/fnm?tab=readme-ov-file#shell-setup )
# # # Fish shell
# # This doesn't work if we open the container using the command `docker exec -it $CONTAINER_NAME /bin/fish` but it works if we open using vscode "open in devcontainer"
# RUN echo -e "\n#Fast Node Manager (fnm)\nfnm env --shell fish | source" >> ~/.config/fish/conf.d/fnm.fish
RUN fnm env --shell fish > ~/.config/fish/conf.d/fnm.fish

# # # Bash shell
RUN echo -e "\n#Fast Node Manager (fnm)\neval \"\$(fnm env --shell bash)\"" >> ~/.bashrc

# # # Zsh shell
RUN echo -e "\n#Fast Node Manager (fnm)\neval \"\$(fnm env --shell zsh)\"" >> ~/.zshrc

# # Adding auto completion for fnm
# # # Fish shell
RUN fnm completions --shell=fish > ~/.config/fish/completions/fnm.fish

# # # Zsh shell
# Set HOME ( or else for root user, it will be /root )
ENV HOME=/home/vscode
RUN fnm completions --shell=zsh > $HOME/_fnm
USER root
RUN mkdir -p /usr/share/zsh/site-functions 
# The below line throw error that: fish: Unknown command: fnm
# RUN fnm completions --shell=zsh > usr/share/zsh/site-functions/_fnm
RUN mv $HOME/_fnm /usr/share/zsh/site-functions/_fnm
USER vscode

# # Download and install Node.js:
RUN fnm install --lts


# # Source fnm env
# # # Zsh shell
# # # # # commenting the below line as the current shell is fish and the below line is required if we want to use fnm in same shell session without restarting
# RUN eval "$(fnm env --shell zsh)" \

# # # Bash shell
# # # # # commenting the below line as the current shell is fish and the below line is required if we want to use fnm in same shell session without restarting
# RUN eval "$(fnm env --shell bash)" \

# # # Fish shell
RUN fnm env --shell fish | source \

    # install yarn
    && npm install -g yarn \

    # Install pnpm
    && npm install -g pnpm

# The above two commands of `npm install` fails with error that npm doesn't exist. Thus, we need to source and run these commands in one go.

#######################################

# Install hardhat
# #  ERR_PNPM_NO_GLOBAL_BIN_DIR  Unable to find the global bin directory
# # Run "pnpm setup" to create it automatically, or set the global-bin-dir setting, or the PNPM_HOME env variable. The global bin directory should be in the PATH
ENV PNPM_HOME=${HOME}/.local/share/pnpm
#  ERROR  The configured global bin directory "/home/vscode/.local/share/pnpm" is not in PATH
ENV PATH=${PATH}:${PNPM_HOME}
RUN pnpm install hardhat -g

# Install bun
# # zsh: bad substitution
# # curl: (23) Failed writing body
# RUN curl -fsSL https://bun.sh/install | zsh
RUN curl -fsSL https://bun.sh/install | bash

# Install uv ( refer: https://docs.astral.sh/uv/getting-started/installation/#standalone-installer )
# # 10.94 no checksums to verify
# # 11.21 installing to /home/vscode/.local/bin
# # 11.21 mv: cannot stat '/tmp/tmp.J9gNOIz5Kd/uv uvx': No such file or directory
# # 11.22 ERROR: command failed: mv /tmp/tmp.J9gNOIz5Kd/uv uvx /home/vscode/.local/bin
# RUN curl -LsSf https://astral.sh/uv/install.sh | zsh
RUN curl -LsSf https://astral.sh/uv/install.sh | bash

# Install tools using uv
# Install slither, slider-lsp (needed for contract explorer), crytic-compile, solc, vyper, mythx, panoramix, napalm-toolbox. napalm-core has to be installed manua
RUN uv tool install slither-analyzer
RUN uv tool install slitherin
RUN uv tool install slither-lsp
RUN uv tool install crytic-compile
RUN uv tool install vyper
RUN uv tool install panoramix-decompiler
RUN uv tool install mythril
RUN uv tool install napalm-toolbox
RUN uv tool install semgrep
RUN uv tool install halmos
RUN uv tool install just-solve-it
RUN uv tool install solc-select
RUN solc-select install 0.4.26 0.5.17 0.6.12 0.7.6 0.8.10 latest && solc-select use latest

# Fetch and install setups
## ityfuzz
RUN curl -fsSL https://ity.fuzz.land/ | zsh
RUN ityfuzzup

# Foundry framework
RUN curl -L https://foundry.paradigm.xyz | zsh
RUN foundryup

# Aderyn
# # The below command is not working probably because they removed it from the repository.
# RUN curl -L https://raw.githubusercontent.com/Cyfrin/aderyn/dev/cyfrinup/install | zsh
RUN curl -L https://raw.githubusercontent.com/Cyfrin/up/main/install | zsh
RUN cyfrinup

## Heimdall
### Replace 'bifrost' call for 'bifrost -B' so it downloads de binary instead of compiling it.
### TODO: ( Check if this is true for current Ubuntu ) Right now this debian uses a glibc version lower than heimdall needs.
RUN curl -fsSL https://get.heimdall.rs | zsh && \
    # source {$HOME}/.cargo/env && \
    {$HOME}/.bifrost/bin/bifrost

# Git clone, compile kind of installations
## Install Medusa
### Set working directory for Medusa operations
WORKDIR ${HOME}/medusa
RUN git clone https://github.com/crytic/medusa {$HOME}/medusa && \
    # For zsh it would be: export LATEST_TAG="$(git ... )"
    set LATEST_TAG (git describe --tags | sed 's/-[0-9]\+-g\w\+$//') && \
    git checkout "$LATEST_TAG" && \
    go build -trimpath -o={$HOME}/.local/bin/medusa -ldflags="-s -w" && \
    chmod 755 {$HOME}/.local/bin/medusa
    #### Return to the home directory and clean up
WORKDIR ${HOME}
RUN rm -rf medusa/

# Copy prebuilt Echidna binary
# COPY --chown=vscode:vscode --from=echidna /usr/local/bin/echidna ${HOME}/.local/bin/echidna
# RUN chmod 755 {$HOME}/.local/bin/echidna
COPY --chmod=755 --chown=vscode:vscode --from=echidna /usr/local/bin/echidna ${HOME}/.local/bin/echidna

# Copy prebuilt Halmos solvers binaries
# COPY --from=halmos \
#     /usr/local/bin/z3 \
#     /usr/local/bin/stp \
#     /usr/local/bin/cvc5 \
#     /usr/local/bin/yices \
#     /usr/local/bin/yices-smt2 \
#     /usr/local/bin/bitwuzla \
#     /usr/local/bin/boolector \
#     ${HOME}/.local/bin/
# RUN chmod 755 {$HOME}/.local/bin/z3 \
#     {$HOME}/.local/bin/stp \
#     {$HOME}/.local/bin/cvc5 \
#     {$HOME}/.local/bin/yices \
#     {$HOME}/.local/bin/yices-smt2 \
#     {$HOME}/.local/bin/bitwuzla \
#     {$HOME}/.local/bin/boolector

COPY --chmod=755 --from=halmos \
    /usr/local/bin/z3 \
    /usr/local/bin/stp \
    /usr/local/bin/cvc5 \
    /usr/local/bin/yices \
    /usr/local/bin/yices-smt2 \
    /usr/local/bin/bitwuzla \
    /usr/local/bin/boolector \
    ${HOME}/.local/bin/


## Add completions for medusa, anvil, cast, forge.
USER root
RUN mkdir -p /usr/share/zsh/site-functions && \
    medusa completion zsh > /usr/share/zsh/site-functions/_medusa && \
    # # This works for bash/zsh but not for fish
    # for tool in anvil cast forge; do \
    #     "$tool" completions zsh > /usr/share/zsh/site-functions/_$tool; \
    #     "$tool" completions fish > ~/.config/fish/completions/_$tool; \
    # done
    # # This works for fish
    for tool in anvil cast forge; \
        "$tool" completions zsh > /usr/share/zsh/site-functions/_$tool; \
        "$tool" completions fish > ~/.config/fish/completions/$tool.fish; \
    end
USER vscode

# Install the Solana CLI ( https://www.anchor-lang.com/docs/installation#install-the-solana-cli )
RUN curl -sSfL https://release.anza.xyz/stable/install | bash
RUN echo -e "\nfish_add_path -a /home/vscode/.local/share/solana/install/active_release/bin\n" >> ~/.config/fish/config.fish

# Install Anchor CLI ( https://www.anchor-lang.com/docs/installation#install-anchor-cli )
RUN cargo install --git https://github.com/coral-xyz/anchor avm --force
RUN avm install latest
RUN avm use latest
# Geting error like this `version `GLIBC_2.38' not found` while running `anchor --version`. The exact error msgs are these two lines:
# /home/vscode/.avm/bin/anchor-0.31.0: /lib/x86_64-linux-gnu/libm.so.6: version `GLIBC_2.38' not found (required by /home/vscode/.avm/bin/anchor-0.31.0)
# /home/vscode/.avm/bin/anchor-0.31.0: /lib/x86_64-linux-gnu/libc.so.6: version `GLIBC_2.39' not found (required by /home/vscode/.avm/bin/anchor-0.31.0)
# And ldd --version gives this output:
    # ldd (Ubuntu GLIBC 2.35-0ubuntu3.9) 2.35
    # Copyright (C) 2022 Free Software Foundation, Inc.
    # This is free software; see the source for copying conditions.  There is NO
    # warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
    # Written by Roland McGrath and Ulrich Drepper.

# Shell Completions ( https://www.anchor-lang.com/docs/installation#shell-completions )
# # Fish shell
RUN anchor completions fish > $HOME/.config/fish/completions/anchor.fish
RUN avm completions fish > $HOME/.config/fish/completions/avm.fish
# # Bash shell
RUN anchor completions bash > $HOME/.config/bash/completions/anchor
RUN avm completions bash > $HOME/.config/bash/completions/avm
# # Zsh shell
RUN anchor completions zsh | sudo tee /usr/local/share/zsh/site-functions/_anchor
RUN avm completions zsh | sudo tee /usr/local/share/zsh/site-functions/_avm

# Clean up
RUN sudo apt-get autoremove -y && sudo apt-get clean -y
