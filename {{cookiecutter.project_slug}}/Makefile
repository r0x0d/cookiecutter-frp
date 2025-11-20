NAME := {{cookiecutter.project_slug}}

all: copr-build srpm sources rust2rpm create-copr-repo

.PHONY: create-copr-repo
create-copr-repo:
	copr create rust-$(NAME) --chroot fedora-rawhide-x86_64

.PHONY: rust2rpm
rust2rpm:
	rust2rpm $(NAME)

.PHONY: sources
sources:
	spectool -g rust-$(NAME).spec

.PHONY: srpm
srpm:
	fedpkg srpm

.PHONY: copr-build
copr-build:
	copr build $(NAME) rust-$(NAME)*.src.rpm
