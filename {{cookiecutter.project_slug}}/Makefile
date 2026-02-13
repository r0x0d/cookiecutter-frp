NAME := {{cookiecutter.project_slug}}
PKG_NAME := {{cookiecutter.pkg_name}}
TOOL2RPM := {{cookiecutter.tool_2_rpm}}
FAS_USERNAME := {{cookiecutter.fas_username}}

FILES_TO_SYNC := $(wildcard *.src.rpm) $(wildcard *.spec)

ifneq ("$(wildcard rust2rpm.toml)","")
FILES_TO_SYNC += rust2rpm.toml
endif

.PHONY: create-copr-repo
create-copr-repo:
	copr create \
		--chroot fedora-43-x86_64 \
		--chroot fedora-44-x86_64 \
		--chroot fedora-rawhide-x86_64 \
		$(NAME)

.PHONY: create-gh-repo
create-gh-repo:
	git init
	git add . && git commit -m "Initial commit for $(NAME)"
	gh repo create $(NAME) \
		--public \
		--disable-wiki \
		--remote origin \
		--source . \
		--push \
		--description "Upstream rpm repository for {{cookiecutter.upstream_repo}}"

.PHONY: spec
spec:
	$(TOOL2RPM) $(PKG_NAME)

.PHONY: sources
sources:
	spectool -g $(NAME).spec

.PHONY: srpm
srpm:
	fedpkg srpm

{% if cookiecutter.build_infra == "copr" %}
.PHONY: build
build: srpm
	copr build $(NAME) $(NAME)*.src.rpm \
		--chroot fedora-43-x86_64 \
		--chroot fedora-44-x86_64 \
		--chroot fedora-rawhide-x86_64
{% else %}
build:
	fedpkg build --srpm --scratch
{% endif %}

.PHONY: sync
sync:
	scp $(FILES_TO_SYNC) $(FAS_USERNAME)@fedorapeople.org:/home/fedora/$(FAS_USERNAME)/public_html/$(NAME)

.PHONY: logs
logs:
	@command -v fuzzytail > /dev/null || { echo >&2 "fuzzytail is not installed. Install with pip install fuzzytail"; }
	fuzzytail watch $(FAS_USERNAME)/$(NAME)
