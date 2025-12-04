NAME := {{cookiecutter.project_slug}}
PKG_NAME := {{cookiecutter.pkg_name}}
TOOL2RPM := {{cookiecutter.tool_2_rpm}}
COPR_USERNAME := {{cookiecutter.copr_username}}

.PHONY: create-copr-repo
create-copr-repo:
	copr create \
		--chroot fedora-43-x86_64 \
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

.PHONY: build
build: srpm
	copr build $(NAME) $(NAME)*.src.rpm \
		--chroot fedora-43-x86_64 \
		--chroot fedora-rawhide-x86_64

.PHONY: logs
logs:
	@command -v fuzzytail > /dev/null || { echo >&2 "fuzzytail is not installed. Install with pip install fuzzytail"; }
	fuzzytail watch $(COPR_USERNAME)/$(NAME)
