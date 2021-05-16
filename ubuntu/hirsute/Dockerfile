FROM ubuntu:hirsute

LABEL maintainer="Victor Song <vsong@protonmail.com>"
ARG CS_USER="martin"
ARG CS_UID="1000"
ARG CS_GID="100"

SHELL ["/bin/bash", "-e", "-u", "-o", "pipefail", "-c"]

USER root

ARG coursier_version="2.0.16"

ENV DEBIAN_FRONTEND noninteractive
# hadolint ignore=DL3008
RUN apt-get -q update && \
    apt-get install -yq --no-install-recommends \
    wget \
    ca-certificates \
    sudo \
    locales && \
    apt-get clean && rm -rf /var/lib/apt/lists/* && \
    echo "en_US.UTF-8 UTF-8" > /etc/locale.gen && \
    locale-gen

ENV SHELL=/bin/bash \
    LC_ALL=en_US.UTF-8 \
    LANG=en_US.UTF-8 \
    LANGUAGE=en_US.UTF-8

COPY fix-permissions /usr/local/bin/fix-permissions
RUN chmod a+rx /usr/local/bin/fix-permissions

# hadolint ignore=SC2016
RUN sed -i 's/^#force_color_prompt=yes/force_color_prompt=yes/' /etc/skel/.bashrc && \
    echo "auth requisite pam_deny.so" >> /etc/pam.d/su && \
    sed -i.bak -e 's/^%admin/#%admin/' /etc/sudoers && \
    sed -i.bak -e 's/^%sudo/#%sudo/' /etc/sudoers && \
    useradd -l -m -s /bin/bash -N -u "$CS_UID" "$CS_USER" && \
    chmod g+w /etc/passwd && \
    fix-permissions "$HOME"

USER $CS_UID

RUN mkdir "/home/$CS_USER/work" && \
    fix-permissions "/home/$CS_USER"

WORKDIR /tmp

RUN wget --quiet -O cs "https://github.com/coursier/coursier/releases/download/v${coursier_version}/cs-x86_64-pc-linux" && \
    chmod +x cs && \
    ./cs install cs && \
    /home/$CS_USER/.local/share/coursier/bin/cs setup -y && \
    /home/$CS_USER/.local/share/coursier/bin/cs install scala3-compiler && \
    /home/$CS_USER/.local/share/coursier/bin/cs install scala3-repl && \
    echo "export PATH=$PATH:/home/$CS_USER/.local/share/coursier/bin" >> "/home/$CS_USER/.bashrc" && \
    rm ./cs

WORKDIR /home/$CS_USER

CMD ["/bin/bash", "--login", "-i"]
