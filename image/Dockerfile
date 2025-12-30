ARG BASE_IMAGE
FROM ${BASE_IMAGE:-debian:trixie}

RUN apt-get update && apt-get install -y --no-install-recommends \
    openssh-server \
    rsync \
    && rm -rf /var/lib/apt/lists/*

RUN sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin prohibit-password/' /etc/ssh/sshd_config \
    && sed -i 's/#PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config \
    && rm /etc/ssh/ssh_host_*_key /etc/ssh/ssh_host_*_key.pub \
    && mkdir -p /run/sshd

RUN <<'EOC'
cat > /usr/local/bin/entrypoint_e7e8494f.sh <<'EOF'
#!/bin/sh
set -e
ssh-keygen -A
if [ -n "$PUBLIC_KEY" ]; then
  mkdir -p /root/.ssh
  echo "$PUBLIC_KEY" > /root/.ssh/authorized_keys
  chmod 700 /root/.ssh
  chmod 600 /root/.ssh/authorized_keys
fi
/usr/sbin/sshd -D &
exec "$@"
EOF
chmod +x /usr/local/bin/entrypoint_e7e8494f.sh
EOC

ENTRYPOINT ["entrypoint_e7e8494f.sh"]

CMD trap : TERM INT; sleep infinity & wait

EXPOSE 22

WORKDIR /root/workspace

ARG RUN
RUN /bin/sh -c "${RUN:-true}"
