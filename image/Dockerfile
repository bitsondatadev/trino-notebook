FROM jupyter/minimal-notebook

LABEL maintainer="Brian Olsen <bits@bitsondata.dev>"

USER root

# Trino dependencies
# Default values can be overridden at build time
# (ARGS are in lower case to distinguish them from ENV)
ARG trino_version="375"

ENV JAVA_HOME=/usr/lib/jvm/zulu11 \
    LANG=en_US.UTF-8 \
    TRINO_VERSION="${trino_version}"

WORKDIR /tmp

RUN set -xeu && \
# Install Zulu OpenJdk 
  apt update && apt install -y gnupg curl && \
  apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 0xB1998361219BD9C9 && \
  curl -O https://cdn.azul.com/zulu/bin/zulu-repo_1.0.0-3_all.deb && \
  apt install ./zulu-repo_1.0.0-3_all.deb && apt update -y && \
  apt install -y zulu-11 && \
# Install Trino binaries
  curl -O "https://repo1.maven.org/maven2/io/trino/trino-server/${TRINO_VERSION}/trino-server-${TRINO_VERSION}.tar.gz" && \
  tar -xf "trino-server-${TRINO_VERSION}.tar.gz" -C /usr/lib && mv /usr/lib/trino-server-${TRINO_VERSION} /usr/lib/trino && \
  rm "trino-server-${TRINO_VERSION}.tar.gz" && \
# Update permissions
  mkdir -p /data/trino && \
  chown -R "${NB_USER}:${NB_GID}" /usr/lib/trino /data/trino && \
# Install Trino CLI
  curl "https://repo1.maven.org/maven2/io/trino/trino-cli/${TRINO_VERSION}/trino-cli-${TRINO_VERSION}-executable.jar" -o /usr/bin/trino && \
  chmod +x /usr/bin/trino && \
# Install Java Kernel
   curl -L https://github.com/SpencerPark/IJava/releases/download/v1.3.0/ijava-1.3.0.zip > ijava-kernel.zip && \
   unzip ijava-kernel.zip -d ijava-kernel && \
   cd ijava-kernel && \
   python3 install.py --sys-prefix && \
# Install Python 3 packages
  pip install trino[sqlalchemy] && \
  mamba install --quiet --yes \
    'matplotlib-base' \
    'scipy' && \
    mamba clean --all -f -y && \
    fix-permissions "${CONDA_DIR}" && \
    fix-permissions "/home/${NB_USER}" && \
# Clean
  apt remove -y gnupg curl && \
  apt clean && rm -rf /var/lib/apt/lists/* && \
  rm -R /tmp/*

# Add default configs
COPY --chown="${NB_USER}:${NB_GID}" etc /etc/trino
COPY --chown="${NB_USER}:${NB_GID}" run-trino "${HOME}"/run-trino

EXPOSE 8080
USER $NB_USER

WORKDIR "${HOME}"
