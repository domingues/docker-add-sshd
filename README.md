# Docker Add SSHd

A minimal base image that adds an SSH server and rsync to any container image.

## Usage

Build a new image using the provided [Dockerfile](image/Dockerfile) with the following options:
 - Set `BASE_IMAGE` to an APT-based image on which `sshd` and `rsync` will be installed (defaults to `debian:trixie`).
 - Optionally set `RUN` to add a custom command to the build instructions.
- `/root/workspace` is the working directory for both build-time `RUN` instruction and the runtime `CMD`.
 - Expose the `sshd` port.
 - Set `PUBLIC_KEY` to your public SSH key; password-based login is disabled.
 - Add a named volume to persist the SSH identity across container recreations.
 - If the base image specifies a `ENTRYPOINT` or `CMD`, add a custom `command` that preserves the original behavior.
 - See additional examples in [`compose-examples.yml`](compose-examples.yml).

### Example
```yaml
services:
  jupyter-notebook:
    image: domingues/jupyter-notebook-sshd:py-3.14
    build:
      context: https://github.com/domingues/docker-add-sshd.git#:image
      args:
        BASE_IMAGE: python:3.14-trixie
        RUN: |
          pip install uv && uv pip install --system \
            notebook \
          && uv cache clean
    ports:
      - "2200:22"
      - "8888:8888"
    environment:
      PUBLIC_KEY: "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIDFWlZ8rT+qLoxs6VBdf6lhB/fZuiWxETqFJBQ4XrXEz"
      JUPYTER_TOKEN: "af4c2615749105e70d0d34700d030b8bc308c44939041078"
    volumes:
      - ssh:/etc/ssh
      - workspace:/root/workspace
    command: ["jupyter", "notebook", "--allow-root", "--ip", "0.0.0.0"]

volumes:
  ssh:
  workspace:
```
