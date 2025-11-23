NAME := {{cookiecutter.project_slug}}
CRATE_NAME := {{cookiecutter.crate_name}}
SPEC_FILE := https://raw.githubusercontent.com/r0x0d/$(NAME)/refs/heads/main/$(NAME).spec

all: copr-build srpm sources rust2rpm create-copr-repo

.PHONY: create-copr-repo
create-copr-repo:
	copr create $(NAME) \
		{% for chroot in cookiecutter.chroots.split(",") %}
		--chroot {{chroot}} \
		{% endfor %}
		--auto-prune "on"

.PHONY: create-gh-repo
create-gh-repo:
	gh repo create {{cookiecutter.project_slug}} \
		--public \
		--disable-wiki \
		--remote origin \
		--description "Upstream rpm repository for {{cookiecutter.upstream_repo_name}}" 

.PHONY: rust2rpm
rust2rpm:
	rust2rpm $(CRATE_NAME)

.PHONY: sources
sources:
	fedpkg sources

.PHONY: srpm
srpm:
	fedpkg srpm

.PHONY: copr-build
copr-build:
	copr build \
	{% for chroot in cookiecutter.chroots.split(",") %}
	--chroot {{chroot}} \
	{% endfor %}	
	$(NAME) $(NAME)*.src.rpm

.PHONY: bugzilla-review
bugzilla-review:
	@if [ ! -f "$(SPEC_FILE)" ]; then \
		echo "Error: Spec file $(SPEC_FILE) not found"; \
		exit 1; \
	fi
	@echo "Creating Bugzilla review request for $(NAME)..."
	@SUMMARY=$$(grep -i '^Summary:' $(SPEC_FILE) | head -n1 | sed 's/^Summary:[[:space:]]*//I'); \
	if [ -z "$$SUMMARY" ]; then \
		echo "Error: Could not find Summary in $(SPEC_FILE)"; \
		exit 1; \
	fi; \
	API_KEY=$$(secret-tool lookup application bugzilla service fedora); \
	if [ -z "$$API_KEY" ]; then \
		echo "Error: Could not retrieve API key from GNOME Keyring"; \
		exit 1; \
	fi; \
	RESPONSE=$$(curl -s -X POST \
		https://bugzilla.redhat.com/rest/bug \
		-H "Content-Type: application/json" \
		-d "{ \
			\"product\": \"Fedora\", \
			\"component\": \"Package Review\", \
			\"summary\": \"Review Request: $(NAME) - $$SUMMARY\", \
			\"version\": \"rawhide\", \
			\"description\": \"Spec URL: $(SPEC_URL)\nSRPM URL: $(SRPM_URL)\nDescription: $$SUMMARY\nFedora Account System Username: r0x0d\", \
			\"api_key\": \"$$API_KEY\" \
		}"); \
	BUG_ID=$$(echo $$RESPONSE | grep -o '"id":[0-9]*' | cut -d: -f2); \
	if [ -n "$$BUG_ID" ]; then \
		echo "✓ Bug created successfully!"; \
		echo "Bug ID: $$BUG_ID"; \
		echo "URL: https://bugzilla.redhat.com/show_bug.cgi?id=$$BUG_ID"; \
	else \
		echo "✗ Failed to create bug"; \
		echo "Response: $$RESPONSE"; \
		exit 1; \
	fi
