PORT ?= 2223
PODMAN ?= $(shell command -v podman)
UID ?= $(shell id -u)
GID ?= $(shell id -g)
HOME ?= $(shell echo $$HOME)
APORTSDIR ?= $(shell echo $$APORTSDIR)

.DEFAULT_GOAL := install

sanity: sanity-podman sanity-uid sanity-gid sanity-aportsdir sanity-home

sanity-podman:
ifeq ($(strip $(PODMAN)),)
	$(error podman must be installed)
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

abuild: abuild.in
	sed -e 's|%%PORT%%|$(PORT)|g' \
		abuild.in >| abuild
	chmod +x abuild

keygen: sanity-home
	@if [ ! -f $(HOME)/.ssh/alpine-dev-env ] || [ ! -f $(HOME)/.ssh/alpine-dev-env.pub ]; then \
		echo y | ssh-keygen -t ed25519 -f $(HOME)/.ssh/alpine-dev-env -q -N ""; \
	fi
	@cp -v $(HOME)/.ssh/alpine-dev-env.pub alpine-dev-env.pub

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

image: sanity-podman
	$(PODMAN) build -t alpine-dev-env .

install: sanity abuild keygen systemd dockerfile image
	install -Dm0755 abuild -t $(HOME)/bin
	install -Dm0644 alpine-ssh.service -t $(HOME)/.config/systemd/user
	rm -f alpine-dev-env.pub
