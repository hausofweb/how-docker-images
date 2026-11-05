ARG RUNNER_VERSION=2.338.2
ARG NODE_MAJOR=26
ARG PNPM_VERSION=12.16.1
ARG AWS_CLI_VERSION=2.22.35
ARG GH_CLI_VERSION=v2.103.3
ARG GH_CLI_SHA256_AMD64=e3bafef96328d1a67de3a968bdfe1cc2ffd58cf6aba8b5e201356897ed06bb3d
ARG GH_CLI_SHA256_ARM64=cac3ff84b60b68e64cca69f31f303210c26320361cc3286b773655d4226b7b06

FROM ghcr.io/actions/actions-runner:${RUNNER_VERSION}

# Re-declare so ARGs are available post-FROM
ARG NODE_MAJOR
ARG PNPM_VERSION
ARG AWS_CLI_VERSION
ARG GH_CLI_VERSION
ARG GH_CLI_SHA256_AMD64
ARG GH_CLI_SHA256_ARM64

USER root

# Misc tools (apt-get before nodesource so lists are fresh)
RUN apt-get update \
	&& apt-get install -y jq unzip \
	&& rm -rf /var/lib/apt/lists/*

# GitHub CLI from its release .deb, the same source Renovate tracks. The apt repo
# drops older versions, so pinning `gh=<version>` there broke every build. The
# apt repo's signature checked the package; the pinned per-arch SHA-256 does that
# here, and Renovate updates the digests with the version.
RUN ARCH=$(dpkg --print-architecture) \
	&& case "$ARCH" in \
		amd64) GH_CLI_SHA256="$GH_CLI_SHA256_AMD64" ;; \
		arm64) GH_CLI_SHA256="$GH_CLI_SHA256_ARM64" ;; \
		*) echo "gh: no pinned digest for architecture $ARCH" >&2; exit 1 ;; \
	esac \
	&& curl -fsSL -o /tmp/gh.deb \
	"https://github.com/cli/cli/releases/download/${GH_CLI_VERSION}/gh_${GH_CLI_VERSION#v}_linux_${ARCH}.deb" \
	&& echo "${GH_CLI_SHA256}  /tmp/gh.deb" | sha256sum -c - \
	&& apt-get update \
	&& apt-get install -y --no-install-recommends /tmp/gh.deb \
	&& rm -f /tmp/gh.deb \
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