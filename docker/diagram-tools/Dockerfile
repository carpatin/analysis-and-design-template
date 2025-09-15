# docker/diagram-tools/Dockerfile
FROM ubuntu:24.04

# Avoid interactive prompts during package installs
ENV DEBIAN_FRONTEND=noninteractive
# Allow Puppeteer to download Chromium later
ENV PUPPETEER_SKIP_DOWNLOAD=false

# -------------------------------------------------------------------
# Base packages:
#  * Java runtime (needed for PlantUML)
#  * Graphviz (diagram rendering backend)
#  * PlantUML
#  * Inkscape & Pandoc for diagram/PDF conversions
#  * Common fonts, including CJK and emoji
#  * Chromium runtime libraries (with Ubuntu 24.04 t64 package names)
# -------------------------------------------------------------------
RUN set -eux; \
  apt-get update; \
  apt-get install -y --no-install-recommends \
    ca-certificates curl gnupg git \
    openjdk-17-jre-headless graphviz plantuml \
    inkscape pandoc \
    fonts-dejavu-core fonts-noto fonts-noto-cjk fonts-noto-color-emoji \
    libatk-bridge2.0-0 libatk1.0-0 libatspi2.0-0 \
    libcups2 libdrm2 libgbm1 libnss3 \
    libx11-xcb1 libxcomposite1 libxdamage1 libxfixes3 libxkbcommon0 \
    libxrandr2 libxrender1 libxshmfence1 \
    "$(apt-cache show libasound2t64 >/dev/null 2>&1 && echo libasound2t64 || echo libasound2)" \
    "$(apt-cache show libgtk-3-0t64  >/dev/null 2>&1 && echo libgtk-3-0t64  || echo libgtk-3-0)"; \
  rm -rf /var/lib/apt/lists/*

# Remove the old package
RUN apt-get purge -y plantuml && rm -rf /var/lib/apt/lists/*
# Install a modern PlantUML
ENV PLANTUML_VERSION=1.2025.7
RUN set -eux; \
    mkdir -p /usr/local/share/plantuml; \
    curl -fL "https://github.com/plantuml/plantuml/releases/download/v${PLANTUML_VERSION}/plantuml-${PLANTUML_VERSION}.jar" \
      -o /usr/local/share/plantuml/plantuml.jar; \
    ln -sf /usr/local/share/plantuml/plantuml.jar /usr/local/bin/plantuml.jar; \
    printf '%s\n' '#!/bin/sh' 'exec java -jar /usr/local/share/plantuml/plantuml.jar "$@"' \
      > /usr/local/bin/plantuml; \
    chmod +x /usr/local/bin/plantuml


# -------------------------------------------------------------------
# Install Node.js 20 LTS
# Needed for Mermaid CLI and Puppeteer
# -------------------------------------------------------------------
RUN mkdir -p /etc/apt/keyrings \
 && curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key \
    | gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg \
 && echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_20.x nodistro main" \
    > /etc/apt/sources.list.d/nodesource.list \
 && apt-get update && apt-get install -y --no-install-recommends nodejs \
 && rm -rf /var/lib/apt/lists/*

# -------------------------------------------------------------------
# Install Mermaid CLI globally and download a stable Chrome
# via @puppeteer/browsers for headless diagram export
# -------------------------------------------------------------------
RUN npm install -g --unsafe-perm @mermaid-js/mermaid-cli @puppeteer/browsers \
 && mkdir -p /opt/puppeteer \
 && npx --yes @puppeteer/browsers install chrome@stable --path /opt/puppeteer \
 && CHROME_BIN="$(find /opt/puppeteer -type f -name chrome -perm -111 | head -n1)" \
 && test -n "$CHROME_BIN" \
 && ln -sf "$CHROME_BIN" /usr/local/bin/chrome

# -------------------------------------------------------------------
# Puppeteer configuration: safer flags for running inside Docker
# -------------------------------------------------------------------
RUN printf '%s\n' \
'{' \
'  "executablePath": "/usr/local/bin/chrome",' \
'  "args": [' \
'    "--no-sandbox",' \
'    "--disable-setuid-sandbox",' \
'    "--font-render-hinting=medium",' \
'    "--disable-dev-shm-usage"' \
'  ]' \
'}' > /etc/puppeteer.json

# Ensure temporary directories exist and are world-writable
RUN mkdir -p /tmp/puppeteer && chmod 1777 /tmp /tmp/puppeteer

# -------------------------------------------------------------------
# Install the complete PlantUML Standard Library
# Makes all extra notations (C4, AWS, etc.) available for !include <>
# -------------------------------------------------------------------
RUN set -eux; \
  git clone --depth=1 https://github.com/plantuml/plantuml-stdlib.git /tmp/plantuml-stdlib; \
  mkdir -p /usr/share/plantuml/stdlib; \
  cp -a /tmp/plantuml-stdlib/* /usr/share/plantuml/stdlib/; \
  rm -rf /tmp/plantuml-stdlib

# Optional symlink for convenience
RUN ln -s /usr/share/plantuml/stdlib /opt/plantuml-stdlib

# Tell PlantUML where to find the stdlib
ENV PLANTUML_INCLUDE_PATH=/usr/share/plantuml/stdlib

# -------------------------------------------------------------------
# Default working directory and command
# -------------------------------------------------------------------
WORKDIR /workspace

# Quick version check and interactive shell when running the container
CMD ["/bin/bash", "-lc", "echo 'Versions:' \
 && java -version || true \
 && plantuml -version || true \
 && node -v && npm -v && mmdc --version || true \
 && inkscape --version || true \
 && pandoc --version | head -n 1 || true \
 && bash"]
