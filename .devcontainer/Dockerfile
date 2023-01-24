FROM mcr.microsoft.com/vscode/devcontainers/base:ubuntu-22.04

ARG USERNAME=vscode
ARG NOVNC_VERSION=1.2.0
ARG WEBSOCKIFY_VERSION=0.9.0
ENV DEBIAN_FRONTEND noninteractive

ARG WIDHT=1024
ARG HEIGHT=768

ENV DISPLAY :0
ENV LANG=en_US.UTF-8 \
  LANGUAGE=en_US.UTF-8 \
  LC_ALL=C.UTF-8 \
  DISPLAY_WIDTH=${WIDHT} \
  DISPLAY_HEIGHT=${HEIGHT}



ARG OCAML_VERSION=4.14.0
ENV OPAM_VERSION  2.1.4
ENV OCAML_VERSION $OCAML_VERSION


RUN apt-get update && apt-get install -y --no-install-recommends build-essential \
  git unzip dosfstools curl wget vim sudo ssh tzdata ca-certificates \
  xauth uuid-dev \
  dbus dbus-x11 alsa-utils pulseaudio pulseaudio-utils mesa-utils x11-apps \
  python3 python-is-python3 python3-distutils python3-numpy python3-pip \
  pkg-config \
  xvfb x11vnc xdotool wget tar supervisor net-tools lxde xterm \
  && apt-get clean -y && rm -rf /var/lib/apt/lists

# Install OPAM
RUN curl -L -o /usr/bin/opam "https://github.com/ocaml/opam/releases/download/$OPAM_VERSION/opam-$OPAM_VERSION-x86_64-Linux" && \
  chmod 755 /usr/bin/opam


# Install noVNC
RUN mkdir -p /usr/local/novnc \
  && curl -k -L -o /tmp/novnc.zip https://github.com/novnc/noVNC/archive/v${NOVNC_VERSION}.zip \
  && unzip /tmp/novnc.zip -d /usr/local/novnc/ \
  && cp /usr/local/novnc/noVNC-${NOVNC_VERSION}/vnc.html /usr/local/novnc/noVNC-${NOVNC_VERSION}/index.html \
  && curl -k -L -o /tmp/websockify.zip https://github.com/novnc/websockify/archive/v${WEBSOCKIFY_VERSION}.zip \
  && unzip /tmp/websockify.zip -d /usr/local/novnc/ \
  && ln -sf /usr/local/novnc/websockify-${WEBSOCKIFY_VERSION} /usr/local/novnc/noVNC-${NOVNC_VERSION}/utils/websockify \
  && rm -rf /tmp/novnc.zip /tmp/websockify.zip

RUN cd /usr/bin && sudo mv lxpolkit lxpolkit.bak

ADD supervisord.conf /etc/supervisor/conf.d/supervisord.conf

USER ${USERNAME}

# OCaml
RUN opam init -a -y --comp $OCAML_VERSION --disable-sandboxing
RUN opam update && \
  opam install -y \
  ocaml-lsp-server \
  dune \
  utop \
  mparser \
  ocamlformat \
  ounit2 \
  qcheck \
  merlin \
  ocamlformat-rpc


RUN mkdir -p /home/${USERNAME}/workspace
WORKDIR /home/${USERNAME}/workspace

USER root
COPY ./.inject_bashrc /root
RUN cat /root/.inject_bashrc >> /home/${USERNAME}/.bashrc
EXPOSE 8085
CMD ["/usr/bin/supervisord"]


