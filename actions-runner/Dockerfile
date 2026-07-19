ARG RUNNER_VERSION=2.335.1
ARG NODE_MAJOR=24
ARG PNPM_VERSION=11.15.0
ARG AWS_CLI_VERSION=2.22.35
ARG GH_CLI_VERSION=2.96.0

FROM ghcr.io/actions/actions-runner:${RUNNER_VERSION}

# Re-declare so ARGs are available post-FROM
ARG NODE_MAJOR
ARG PNPM_VERSION
ARG AWS_CLI_VERSION
ARG GH_CLI_VERSION

USER root

# Misc tools (apt-get before nodesource so lists are fresh)
RUN apt-get update \
	&& apt-get install -y jq unzip \
	&& rm -rf /var/lib/apt/lists/*

RUN (type -p wget >/dev/null || (apt-get update && apt-get install -y wget)) \
	&& mkdir -p -m 755 /etc/apt/keyrings \
	&& out=$(mktemp) && wget -nv -O$out https://cli.github.com/packages/githubcli-archive-keyring.gpg \
	&& cat $out | tee /etc/apt/keyrings/githubcli-archive-keyring.gpg > /dev/null \
	&& chmod go+r /etc/apt/keyrings/githubcli-archive-keyring.gpg \
	&& mkdir -p -m 755 /etc/apt/sources.list.d \
	&& echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | tee /etc/apt/sources.list.d/github-cli.list > /dev/null \
	&& apt-get update \
	&& apt-get install -y gh=${GH_CLI_VERSION} \
	&& rm -f $out \
	&& rm -rf /var/lib/apt/lists/*

# Node.js
RUN curl -fsSL https://deb.nodesource.com/setup_${NODE_MAJOR}.x | bash - \
	&& apt-get install -y nodejs \
	&& rm -rf /var/lib/apt/lists/*

# pnpm
RUN npm install -g pnpm@${PNPM_VERSION}

# AWS CLI (arch-aware, pinned)
RUN ARCH=$(uname -m) \
	&& if [ "$ARCH" = "aarch64" ]; then \
	URL="https://awscli.amazonaws.com/awscli-exe-linux-aarch64-${AWS_CLI_VERSION}.zip"; \
	else \
	URL="https://awscli.amazonaws.com/awscli-exe-linux-x86_64-${AWS_CLI_VERSION}.zip"; \
	fi \
	&& curl -sSf "$URL" -o awscliv2.zip \
	&& unzip -q awscliv2.zip \
	&& ./aws/install \
	&& rm -rf awscliv2.zip aws/

USER runner
ENTRYPOINT ["/home/runner/run.sh"]