# Copyright 2017-2020 Authors of Cilium
# SPDX-License-Identifier: Apache-2.0

include ../Makefile.defs
include ../Makefile.quiet

.PHONY: default clean builder-image cmdref epub latex html run-server stop-server

default: html

clean:
	-$(QUIET)rm -rf _build

builder-image: Dockerfile requirements.txt
	$(QUIET)tar c requirements.txt Dockerfile \
	  | $(CONTAINER_ENGINE) build --tag cilium/docs-builder -

DOCKER_RUN := $(CONTAINER_ENGINE_FULL) container run --rm \
		--workdir /src/Documentation \
		--volume $(CURDIR)/..:/src \
		--user "$(shell id -u):$(shell id -g)" \
			cilium/docs-builder

update-cmdref: builder-image
	@$(ECHO_GEN)cmdref
	-$(QUIET)rm -rf cmdref/cilium*.md
	$(QUIET)$(DOCKER_RUN) ./update-cmdref.sh

check: builder-image update-cmdref
	@$(ECHO_CHECK) cmdref
	$(QUIET)$(DOCKER_RUN) ./check-cmdref.sh
	@$(ECHO_CHECK) examples
	$(QUIET)$(DOCKER_RUN) ./check-examples.sh

ifeq ($(V),0)
SPHINX_OPTS += -q
endif

epub latex html: builder-image
	@$(ECHO_GEN)_api
	$(QUIET)cp -r ../api _api
	@$(ECHO_GEN)_build/$@
	$(QUIET)$(DOCKER_RUN) ./check-build.sh $(@) $(SPHINX_OPTS)

DOCS_PORT = 9081

run-server: stop-server
	# Work around Docker issue where this directory is created as root in
	# the `--volume` parameter below. See also GH-10869.
	$(QUIET)mkdir -p $(CURDIR)/_build/html/
	$(QUIET)$(CONTAINER_ENGINE) container run --rm \
		--detach \
		--interactive \
		--tty \
		--name docs-cilium \
		--publish $(DOCS_PORT):80 \
		--volume $(CURDIR)/_build/html/:/usr/local/apache2/htdocs/ \
			httpd:2.4
	@echo "$$(tput setaf 2)Running at http://localhost:$(DOCS_PORT)$$(tput sgr0)"

stop-server:
	-$(QUIET)$(CONTAINER_ENGINE) container rm --force --volumes docs-cilium
