ARG VARIANT="bookworm"
FROM mcr.microsoft.com/vscode/devcontainers/rust:1-${VARIANT} AS builder

# Install cargo tools
RUN curl -L "https://github.com/cargo-bins/cargo-binstall/releases/latest/download/cargo-binstall-x86_64-unknown-linux-musl.tgz" | tar -xz -C /usr/local/cargo/bin

RUN cargo binstall --no-confirm \
    cargo-audit \
    cargo-nextest \
    cargo-llvm-cov \
    cargo-machete \
    sccache \
    wasm-pack@0.13.1 \
    grcov 

RUN cargo install dioxus-cli
    
# Install Bun
RUN curl -fsSL https://bun.sh/install | bash 

# Install kubectl
RUN curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl" \
     && chmod +x kubectl \
     && mv kubectl /usr/local/bin/

# Install k9s for Linux (x86_64)
RUN curl -LO "https://github.com/derailed/k9s/releases/latest/download/k9s_Linux_amd64.tar.gz" \
     && tar -xzf k9s_Linux_amd64.tar.gz \
     && chmod +x k9s \
     && mv k9s /usr/local/bin/ \
     && rm k9s_Linux_amd64.tar.gz

# Install helm for Linux (x86_64)
ARG HELM_VERSION="3.16.2"
RUN curl -LO "https://get.helm.sh/helm-v${HELM_VERSION}-linux-amd64.tar.gz" \
    && tar -xzf helm-v${HELM_VERSION}-linux-amd64.tar.gz \
    && chmod +x linux-amd64/helm \
    && mv linux-amd64/helm /usr/local/bin/ \
    && rm -rf linux-amd64 helm-v${HELM_VERSION}-linux-amd64.tar.gz

# Install helmfile for Linux (x86_64)
ARG HELMFILE_CLI_VERSION="0.169.1"
RUN curl -LO "https://github.com/helmfile/helmfile/releases/download/v${HELMFILE_CLI_VERSION}/helmfile_${HELMFILE_CLI_VERSION}_linux_amd64.tar.gz" \
    && tar -xzf helmfile_${HELMFILE_CLI_VERSION}_linux_amd64.tar.gz \
    && chmod +x helmfile \
    && mv helmfile /usr/local/bin/ \
    && rm helmfile_${HELMFILE_CLI_VERSION}_linux_amd64.tar.gz

# Install Stripe CLI for Linux (x86_64)
ARG STRIPE_CLI_VERSION="1.22.0"
RUN curl -LO "https://github.com/stripe/stripe-cli/releases/download/v${STRIPE_CLI_VERSION}/stripe_${STRIPE_CLI_VERSION}_linux_x86_64.tar.gz" \
    && tar -xzf stripe_${STRIPE_CLI_VERSION}_linux_x86_64.tar.gz \
    && chmod +x stripe \
    && mv stripe /usr/local/bin/ \
    && rm stripe_${STRIPE_CLI_VERSION}_linux_x86_64.tar.gz

# Install grpcurl for Linux (x86_64)
ARG GRPCURL_VERSION="1.9.1"
RUN curl -LO "https://github.com/fullstorydev/grpcurl/releases/download/v${GRPCURL_VERSION}/grpcurl_${GRPCURL_VERSION}_linux_x86_64.tar.gz" \
     && tar -xzf grpcurl_${GRPCURL_VERSION}_linux_x86_64.tar.gz \
     && chmod +x grpcurl \
     && mv grpcurl /usr/local/bin/ \
     && rm grpcurl_${GRPCURL_VERSION}_linux_x86_64.tar.gz

# Slim runtime image
FROM mcr.microsoft.com/devcontainers/base:${VARIANT}

# Metadata
LABEL maintainer="ETO GRUPPE TECHNOLOGIES GmbH <support@cawaena.com>"
LABEL description="Dev container for Cawaena development with tooling"

# Copy only necessary files from the builder stage
COPY --from=builder /usr/local/bin /usr/local/bin
COPY --from=builder /usr/local/cargo /home/vscode/.cargo
COPY --from=builder /usr/local/rustup /home/vscode/.rustup
COPY --from=builder /root/.bun/bin/bun /usr/local/bin
COPY --from=builder /root/.bun/bin/bunx /usr/local/bin

# Add docker client
COPY --from=docker:latest /usr/local/bin/docker /usr/local/bin/

# Set user permissions
RUN chown -R vscode:vscode /home/vscode/.cargo

# Install runtime dependencies
ARG CLANG_VERSION="16"
RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
    && apt-get -y install --no-install-recommends \
    libc6 libssl-dev musl-tools clang-${CLANG_VERSION} lld-${CLANG_VERSION} \
    pkg-config  protobuf-compiler libprotobuf-dev \
    nodejs gawk bison  \
    && rm -rf /var/lib/apt/lists/*

# Set default user
USER vscode

# Configure PATH
ENV CARGO_HOME="/home/vscode/.cargo"
ENV RUSTUP_HOME="/home/vscode/.rustup"
ENV PATH="/usr/local/bin:/home/vscode/.cargo/bin:/home/vscode/.cargo/bin:/home/vscode/.bun/bin:${PATH}"

# Install Rust targets and components
RUN rustup default stable \
    && rustup component add rustfmt clippy llvm-tools-preview \
    && rustup target add x86_64-unknown-linux-musl wasm32-unknown-unknown 

# Clean cargo home to reduce size
RUN rm -rf /usr/local/cargo/registry/*    

# Install JavaScript tools
RUN bun install -g npm ts-node typescript pnpm

# Set entrypoint
CMD ["/bin/bash"]
