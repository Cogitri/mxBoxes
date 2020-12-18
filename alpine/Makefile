PORT ?= 2223
PODMAN ?= $(shell command -v podman)
PODMAN_BUILD ?= $(shell command -v podman)
SSH_KEYGEN ?= $(shell command -v ssh-keygen)
UID ?= $(shell id -u)
GID ?= $(shell id -g)
HOME ?= $(shell echo $$HOME)
APORTSDIR ?= $(shell echo $$APORTSDIR)

.DEFAULT_GOAL := install

sanity: sanity-ssh-keygen \
		sanity-podman \
		sanity-podman-build \
		sanity-uid \
		sanity-gid \
		sanity-aportsdir \
		sanity-home

sanity-podman:
ifeq ($(strip $(PODMAN)),)
	$(error podman to run in the systemd system service could not be found, set it with PODMAN=)
endif

sanity-podman-build:
ifeq ($(strip $(PODMAN_BUILD)),)
	$(error podman used to build the image could not be found, set it with PODMAN_BUILD=)
endif

sanity-uid:
ifeq ($(strip (UID)),)
	$(error could not get user uid via "id -u")
endif

sanity-gid:
ifeq ($(strip $(GID)),)
	$(error could not get user gid via "id -g")
endif

sanity-aportsdir:
ifeq ($(strip $(APORTSDIR)),)
	$(error APORTSDIR is not set, must be absolute path to aports)
endif

sanity-home:
ifeq ($(strip $(HOME)),)
	$(error HOME is not set, it must be set to the home directory)
endif

sanity-ssh-keygen:
ifeq ($(strip $(SSH_KEYGEN)),)
	$(error ssh-keygen not installed, pass path to ssh-keygen binary via SSH_KEYGEN=)
endif

abuild: abuild.in
	sed -e 's|%%PORT%%|$(PORT)|g' \
		-e 's|%%APORTSDIR%%|$(APORTSDIR)|g' \
		abuild.in >| abuild
	chmod +x abuild

keygen: sanity-ssh-keygen sanity-home
	@if [ ! -f $(HOME)/.ssh/alpine-dev-env ] || [ ! -f $(HOME)/.ssh/alpine-dev-env.pub ]; then \
		echo y | $(SSH_KEYGEN) -t ed25519 -f $(HOME)/.ssh/alpine-dev-env -q -N ""; \
	fi
	@if [ ! -f $(HOME)/.ssh/authorized_keys ]; then \
		cat $(HOME)/.ssh/alpine-dev-env.pub >> $(HOME)/.ssh/authorized_keys; \
	elif ! grep -q "$$(cat $(HOME)/.ssh/alpine-dev-env.pub)" $(HOME)/.ssh/authorized_keys; then \
		cat $(HOME)/.ssh/alpine-dev-env.pub >> $(HOME)/.ssh/authorized_keys; \
	fi

systemd: sanity-podman sanity-aportsdir sanity-home alpine-ssh.service.in
	sed -e 's|%%PORT%%|$(PORT)|g' \
		-e 's|%%PODMAN%%|$(PODMAN)|g' \
		-e 's|%%APORTSDIR%%|$(APORTSDIR)|g' \
		-e 's|%%HOME%%|$(HOME)|g' \
		alpine-ssh.service.in >| alpine-ssh.service

dockerfile: sanity-uid sanity-gid Dockerfile.in
	sed	-e 's|%%PORT%%|$(PORT)|g' \
		-e 's|%%UID%%|$(UID)|g' \
		-e 's|%%GID%%|$(GID)|g' \
		Dockerfile.in >| Dockerfile

image: sanity-podman-build
	$(PODMAN_BUILD) build -t alpine-dev-env .

install: sanity abuild keygen systemd dockerfile image
	install -Dm0755 abuild -t $(HOME)/bin
	install -Dm0644 alpine-ssh.service -t $(HOME)/.config/systemd/user
