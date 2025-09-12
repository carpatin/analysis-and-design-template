# docker/diagram-tools/Dockerfile
FROM ubuntu:24.04

ENV DEBIAN_FRONTEND=noninteractive
ENV PUPPETEER_SKIP_DOWNLOAD=false

# Base packages + PlantUML/Graphviz + Inkscape + Pandoc + fonts + Chromium deps
# Handle Ubuntu 24.04 t64 library renames (libasound2 -> libasound2t64, libgtk-3-0 -> libgtk-3-0t64)
RUN set -eux; \
  apt-get update; \
  apt-get install -y --no-install-recommends \
    ca-certificates curl gnupg \
    openjdk-17-jre-headless graphviz plantuml \
    inkscape pandoc \
    fonts-noto fonts-noto-cjk fonts-noto-color-emoji \
    libatk-bridge2.0-0 libatk1.0-0 libatspi2.0-0 \
    libcups2 libdrm2 libgbm1 libnss3 \
    libx11-xcb1 libxcomposite1 libxdamage1 libxfixes3 libxkbcommon0 \
    libxrandr2 libxrender1 libxshmfence1 \
    $(apt-cache show libasound2t64 >/dev/null 2>&1 && echo libasound2t64 || echo libasound2) \
    $(apt-cache show libgtk-3-0t64  >/dev/null 2>&1 && echo libgtk-3-0t64  || echo libgtk-3-0); \
  rm -rf /var/lib/apt/lists/*

# Node.js 20 LTS (Mermaid CLI needs modern Node)
RUN mkdir -p /etc/apt/keyrings \
 && curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key \
    | gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg \
 && echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_20.x nodistro main" \
    > /etc/apt/sources.list.d/nodesource.list \
 && apt-get update && apt-get install -y --no-install-recommends nodejs \
 && rm -rf /var/lib/apt/lists/*

# Mermaid CLI (global). --unsafe-perm lets Puppeteer postinstall (Chromium) run as root
RUN npm install -g --unsafe-perm @mermaid-js/mermaid-cli

# Install the official browser binary via @puppeteer/browsers
# We put it in /opt/puppeteer and make a stable symlink to the chrome executable.
RUN npm install -g @puppeteer/browsers \
 && mkdir -p /opt/puppeteer \
 && npx --yes @puppeteer/browsers install chrome@stable --path /opt/puppeteer \
 && CHROME_BIN="$(find /opt/puppeteer -type f -name chrome -perm -111 | head -n1)" \
 && test -n "$CHROME_BIN" \
 && ln -sf "$CHROME_BIN" /usr/local/bin/chrome

# Puppeteer config: force flags and executable path
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

# Ensure temp dirs exist and are writable for any uid
RUN mkdir -p /tmp/puppeteer && chmod 1777 /tmp /tmp/puppeteer

# Add git and install C4-PlantUML into a stdlib folder so !include <C4/...> works
RUN apt-get update && apt-get install -y --no-install-recommends git && rm -rf /var/lib/apt/lists/*
RUN set -eux; \
  mkdir -p /opt/plantuml-stdlib; \
  git clone --depth=1 https://github.com/plantuml-stdlib/C4-PlantUML.git /opt/plantuml-stdlib/C4-PlantUML; \
  mkdir -p /opt/plantuml-stdlib/C4; \
  cp /opt/plantuml-stdlib/C4-PlantUML/C4*.puml /opt/plantuml-stdlib/C4/; \
  rm -rf /opt/plantuml-stdlib/C4-PlantUML/.git

# Add git and install C4-PlantUML into PlantUML's stdlib
RUN apt-get update && apt-get install -y --no-install-recommends git && rm -rf /var/lib/apt/lists/*
RUN set -eux; \
  git clone --depth=1 https://github.com/plantuml-stdlib/C4-PlantUML.git /tmp/C4-PlantUML; \
  mkdir -p /usr/share/plantuml/stdlib/C4; \
  cp /tmp/C4-PlantUML/C4*.puml /usr/share/plantuml/stdlib/C4/; \
  rm -rf /tmp/C4-PlantUML

# (Optional) also keep an env include path for good measure
ENV PLANTUML_INCLUDE_PATH=/usr/share/plantuml/stdlib


# Optional: quick versions check when you run the image interactively
CMD ["/bin/bash", "-lc", "echo 'Versions:' \
 && java -version || true \
 && plantuml -version || true \
 && node -v && npm -v && mmdc --version || true \
 && inkscape --version || true \
 && pandoc --version | head -n 1 || true \
 && bash"]
