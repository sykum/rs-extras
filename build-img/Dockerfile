# syntax=docker/dockerfile:1

FROM alpine:3 AS rootfs-stage

# install packages
RUN \
  apk add --no-cache \
    bash \
    curl \
    jq \
    tzdata \
    xz

# grab latest rootfs
RUN \
  echo "**** download/extract rootfs ****" && \
  curl -o \
    /rootfs.tar.xz -L \
    http://os.archlinuxarm.org/os/ArchLinuxARM-aarch64-latest.tar.gz && \
  mkdir /root-out && \
  tar xf \
    /rootfs.tar.xz -C \
    /root-out

# pacstrap stage
FROM scratch AS pacstrap-stage
COPY --from=rootfs-stage /root-out/ /

RUN \
  mkdir -m 0755 -p \
    /root-out/var/{cache/pacman/pkg,lib/pacman,log} \
    /root-out/{dev,run,etc} && \
  mkdir -m 1777 -p \
    /root-out/tmp && \
  mkdir -m 0555 -p \
    /root-out/{sys,proc} && \
  sed -i 's/DownloadUser = alpm//g' /etc/pacman.conf && \
  pacman-key --init && \
  pacman-key --populate archlinuxarm && \
  pacman -r /root-out -Sy --noconfirm \
    archlinuxarm-keyring \
    bash \
    busybox \
    catatonit \
    coreutils \
    inetutils \
    curl \
    findutils \
    gawk \
    grep \
    gzip \
    jq \
    less \
    netcat \
    pacman \
    procps-ng \
    sed \
    shadow \
    tar \
    tzdata \
    util-linux \
    which && \
  cp -a /etc/pacman.conf /root-out/etc/pacman.conf && \
  cp -a /etc/pacman.d/mirrorlist /root-out/etc/pacman.d/mirrorlist && \
  cp -a /etc/pacman.d/gnupg /root-out/etc/pacman.d/gnupg && \
  rm /root-out/var/lib/pacman/sync/*

# runtime stage
FROM scratch
COPY --from=pacstrap-stage /root-out/ /

# environment variables
ENV PS1="$(whoami)@$(hostname):$(pwd)\\$ " \
  HOME="/root" \
  TERM="xterm"

RUN \
  echo "**** configure pacman ****" && \
  locale-gen && \
  echo "**** configure locale ****" && \
  echo "en_US.UTF-8 UTF-8" > /etc/locale.gen && \
  locale-gen && \
  echo "**** cleanup ****" && \
  chmod 755 \
    /etc \
    /usr \
    /usr/bin && \
  rm -rf \
    /tmp/* \
    /var/cache/pacman/pkg/* \
    /var/lib/pacman/sync/*

RUN \
  echo "**** install build dependencies ****" && \
  pacman -Sy --noconfirm base-devel devtools && \
  useradd -m -G wheel builder && \
  echo "builder ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers

USER builder
WORKDIR /home/builder

ENTRYPOINT ["/bin/bash"]