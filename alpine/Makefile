PORT ?= 2223
PODMAN ?= $(shell echo $${PODMAN:-/usr/bin/podman})
USER ?= $(shell id -u -n)
UID ?= $(shell id -u)
GID ?= $(shell id -g)
HOME ?= $$HOME
APORTSDIR ?= $$APORTSDIR

.PHONY: install
all: sanity abuild dockerfile keygen systemd

sanity:
	@[ "$$APORTSDIR" ] || ( \
		$(error APORTSDIR is not set, must be absolute path to aports)
	)

abuild: abuild.in
	sed -e 's|%%PORT%%|$(PORT)|g' \
		abuild.in >| abuild
	chmod +x abuild

keygen:
	@if [ ! -f $$HOME/.ssh/alpine-dev-env ] || [ ! -f $$HOME/.ssh/alpine-dev-env.pub ]; then \
		echo y | ssh-keygen -t ed25519 -f $$HOME/.ssh/alpine-dev-env -q -N ""; \
	fi
	@cp -v $$HOME/.ssh/alpine-dev-env.pub alpine-dev-env.pub

systemd: alpine-ssh.service.in
	sed -e 's|%%PORT%%|$(PORT)|g' \
		-e 's|%%PODMAN%%|$(PODMAN)|g' \
		-e 's|%%APORTSDIR%%|$(APORTSDIR)|g' \
		-e 's|%%HOME%%|$(HOME)|g' \
		alpine-ssh.service.in >| alpine-ssh.service

dockerfile: Dockerfile.in
	sed	-e 's|%%PORT%%|$(PORT)|g' \
		-e 's|%%UID%%|$(UID)|g' \
		-e 's|%%GID%%|$(GID)|g' \
		Dockerfile.in >| Dockerfile

image: all
	$(PODMAN) build -t alpine-dev-env .

install: image
	install -Dm0755 abuild -t $(HOME)/bin
	install -Dm0644 alpine-ssh.service -t $$HOME/.config/systemd/user
	rm -f alpine-dev-env.pub
