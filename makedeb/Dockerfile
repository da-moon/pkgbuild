# syntax=docker/dockerfile:1.3-labs
#-*-mode:dockerfile;indent-tabs-mode:nil;tab-width:2;coding:utf-8-*-
# vim: filetype=dockerfile softtabstop=2 tabstop=2 shiftwidth=2 fenc=utf-8 fileformat=unix expandtab
FROM "debian:bullseye"
SHELL ["/bin/bash", "-o", "pipefail", "-c"]
# ─── LOCALE SETUP ───────────────────────────────────────────────────────────────
USER "root"
ENV LANG='en_US.UTF-8' LANGUAGE='en_US:en' LC_ALL='en_US.UTF-8' TIMEZONE='UTC'
ENV DEBIAN_FRONTEND="noninteractive"
ENV TERM=xterm
# [ NOTE ] => locales-all is a necessary package for locale setup
RUN \
    apt-get update -y \
    && apt-get install -y "locales" "locales-all" \
    && export LANGUAGE="${LANGUAGE}"  \
    && export LANG="${LANG}" \
    && export LC_ALL="${LC_ALL}" \
    && locale-gen "${LANG}" \
    && dpkg-reconfigure --frontend noninteractive locales \
    && update-locale LANG="${LANG}"

# ─── ESSENTIAL PACKAGES ─────────────────────────────────────────────────────────
USER "root"
RUN \
  apt-get update -q \
  && apt-get install -yq  --no-install-recommends \
    "wget" \
    "perl" \
    "git" \
    "gnupg" \
    "ca-certificates" \
    "openssh-client" \
    "sudo" ;

# ─── SUDO SETUP ─────────────────────────────────────────────────────────────────
USER "root"
RUN \
  sed -i \
  -e '/%sudo.*NOPASSWD:ALL/d' \
  -e '/%sudo\s\+ALL=(ALL\(:ALL\)\?)\s\+ALL/d' \
  "/etc/sudoers" \
  && ( \
    echo '%sudo ALL=(ALL) ALL' ; \
    echo '%sudo ALL=(ALL) NOPASSWD: ALL' ; \
  ) | tee -a "/etc/sudoers" > /dev/null
# 
#  ──── MAKEDEB INSTALL ───────────────────────────────────────────────
# 
USER "root"
COPY <<-"EOT" "/etc/apt/sources.list.d/makedeb.list"
deb [signed-by=/usr/share/keyrings/makedeb-archive-keyring.gpg arch=all] https://proget.hunterwittenborn.com/ makedeb main
EOT
RUN \
  wget -qO - 'https://proget.hunterwittenborn.com/debian-feeds/makedeb.pub' \
    | gpg --dearmor \
    | tee "/usr/share/keyrings/makedeb-archive-keyring.gpg" > /dev/null \
  && apt-get update -q \
  && apt-get install -yq  --no-install-recommends "makedeb"
#
#  ──── USER CREATION ─────────────────────────────────────────────────
#
USER "root"
ENV USER="makedeb"
ENV UID="1000"
ENV HOME="/home/${USER}"
RUN \
  useradd \
  --no-log-init \
  --user-group \
  --create-home \
  --home-dir "${HOME}" \
  --uid "${UID}" \
  --groups sudo \
  --shell "/bin/bash" \
  --password \
  $(perl \
    -e 'print crypt($ARGV[0], "password")' \
      "${USER}_${UID}" 2>/dev/null) \
  "${USER}"  \
  && usermod -aG sudo,root "${USER}" ;
# 
#  ──── STORING SSH KEY ───────────────────────────────────────────────
# 
USER "root"
RUN \
  --mount=type=secret,id=id_rsa \
  cp "/run/secrets/id_rsa" "/tmp/id_rsa" \
  && mkdir "/workspace" "/workspace-git" \
  && chown -R  "$(id -u "${USER}"):$(id -g "${USER}")" \
    "/workspace" \
    "/workspace-git" \
    "/tmp/id_rsa"
# 
#  ──── SSH SETUP ─────────────────────────────────────────────────────
# 
USER "${USER}"
RUN \
  mkdir -p "${HOME}/.ssh" \
  && chmod 700 "${HOME}/.ssh" \
  && mv "/tmp/id_rsa" "${HOME}/.ssh/id_rsa" \
  && chmod 600 "${HOME}/.ssh/id_rsa"

COPY --chown=makedeb --chmod=0644 <<-"EOT" "/home/makedeb/.ssh/config"
Host mpr.hunterwittenborn.com
  IdentityFile ~/.ssh/id_rsa
  User mpr
  IdentitiesOnly yes
  StrictHostKeyChecking no
  MACs hmac-sha2-512
  UserKnownHostsFile=/dev/null
EOT
ARG PKG_NAME
ARG GIT_EMAIL="the.alfador@pm.me"
ARG GIT_NAME="alfador"
RUN \
  git config --global user.email "${GIT_EMAIL}" \
  && git config --global user.name "${GIT_NAME}" \
  && git clone "ssh://mpr@mpr.hunterwittenborn.com/${PKG_NAME}.git" "/workspace-git";

COPY --chown=makedeb "./${PKG_NAME}" "/workspace"

WORKDIR "/workspace"
RUN \
  makedeb --printsrcinfo | tee .SRCINFO ; \
  [ ! -r "/workspace-git/PKGBUILD" ] \
  && find "/workspace-git" -mindepth 1 -maxdepth 1 -type f -delete ; \
  while read file;do \
  ln -f "${file}" "/workspace-git/$(basename "${file}")" ; \
  done < <(find "/workspace" -type f) ;
# 
#  ──── CLEANUP ───────────────────────────────────────────────────────
# 
USER "root"
  # apt-get remove -y --purge \
  #   "perl" \
  #   "git" \
  #   "wget" \
  #   "gnupg" \
  #   "ca-certificates" \
RUN \
  apt-get autoremove -y \
  && apt-get clean -y \
  && rm -rf \
     "/var/cache/debconf/"* \
     "/var/lib/apt/lists/"* \
     "/tmp/"* \
     "/var/tmp/"* \
  && apt-get update
#  ────────────────────────────────────────────────────────────────────
USER "${USER}"
VOLUME ["workspace"]
WORKDIR "/workspace"
ENTRYPOINT ["/bin/bash"]