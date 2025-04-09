# 
FROM openjdk:11 AS build-env

LABEL MAINTAINER="TheHive Project <support@thehive-project.org>"

ARG THEHIVE_VERSION=main

RUN apt update && \
  apt install -y --no-install-recommends \
  ca-certificates \
  apt-transport-https && \
  curl -sL https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.3/install.sh | bash - && \
  export NVM_DIR="${HOME}/.nvm" && \
  . "$NVM_DIR/nvm.sh" && \
  nvm install 14 && \
  apt-get install -y git && \
  npm install -g grunt-cli \
  bower && \
  git -c advice.detachedHead=false \
  clone \
  --branch=$THEHIVE_VERSION \
  --depth=1 \
  --recurse-submodules \
  https://github.com/vanvy102/TheHive.git && \
  echo '{"allow_root": true}' > /root/.bowerrc && \
  cd TheHive && \
  # Build the frontend (UI) in the correct subdirectory
  cd frontend && \
  export NVM_DIR="/root/.nvm" && \
  . "$NVM_DIR/nvm.sh" && \
  npm install --legacy-peer-deps && \
  bower install --allow-root && \
  cd .. && \
  # Grant executable permission to sbt script

  # Build the backend (Scala) with verbose output
  ./sbt -v clean stage && \
  mv /TheHive/target/universal/stage /opt/thehive && \
  mv /TheHive/package/docker/entrypoint /opt/thehive/entrypoint && \
  mkdir /var/log/thehive && \
  # Clean up after both builds are done
  apt-get purge -y git && \
  rm -rf /TheHive \
  /root/* \
  /root/.nvm \
  /root/.m2 \
  /root/.ivy2 \
  /root/.sbt \
  /var/lib/apt/lists/*

# Runtime stage
FROM openjdk:11
COPY --from=build-env /opt/thehive /opt/thehive
COPY --from=build-env /var/log/thehive /var/log/thehive

RUN apt update && \
  apt upgrade -y && \
  apt autoclean -y -q && \
  apt autoremove -y -q && \
  rm -rf /var/lib/apt/lists/* && \
  ( type groupadd 1>/dev/null 2>&1 && \
  groupadd -g 1000 thehive || \
  addgroup -g 1000 -S thehive ) && \
  ( type useradd 1>/dev/null 2>&1 && \
  useradd --system --uid 1000 --gid 1000 thehive || \
  adduser -S -u 1000 -G thehive thehive ) && \
  mkdir /etc/thehive && \
  cp /opt/thehive/conf/logback.xml /etc/thehive/logback.xml && \
  chown -R root:root /opt/thehive && \
  touch /var/log/thehive/application.log && \
  chown -R thehive:thehive /var/log/thehive /etc/thehive && \
  chmod +x /opt/thehive/entrypoint

USER thehive

EXPOSE 9000

WORKDIR /opt/thehive

ENTRYPOINT ["/opt/thehive/entrypoint"]
