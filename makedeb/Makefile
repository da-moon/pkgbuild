# vim: ft=make syntax=make softtabstop=2 tabstop=2 shiftwidth=2 fenc=utf-8 expandtab

# 
#  ──── RECURSIVE WILDCARD ───────────────────────────────────────────
# 
define rwildcard
	$(wildcard $1$2) $(foreach d,$(wildcard $1*),$(call rwildcard,$d/,$2))
endef
ifeq ($(OS),Windows_NT)
  CLEAR = cls
  LS = dir
  TOUCH =>> 
  RM = del /F /Q
  CPF = copy /y
  RMDIR = -RMDIR /S /Q
  MKDIR = -mkdir
  ERRIGNORE = 2>NUL || (exit 0)
  SEP=\\
  DEVNUL := NUL
  WHICH := where
else
  CLEAR = clear
  LS = ls
  TOUCH = touch
  CPF = cp -f
  RM = rm -rf 
  RMDIR = rm -rf 
  MKDIR = mkdir -p
  ERRIGNORE = 2>/dev/null
  SEP=/
  DEVNUL = /dev/null
  WHICH = which
endif

ifeq ($(UNAME),Darwin)
	OS_X  := true
	SHELL := /opt/local/bin/bash
else ifneq (,$(wildcard /etc/redhat-release))
	OS_RHEL := true
	SHELL := /bin/bash
else ifeq ($(findstring cmd.exe,$(SHELL)),cmd.exe)
	OS_WIN  := true
else
	OS_DEB  := true
	SHELL := /bin/bash
endif
nullstring :=
space := $(nullstring) 
PWD ?= $(realpath $(dir $(lastword $(MAKEFILE_LIST))))
PSEP = $(strip $(SEP))
SELF_DIR := $(dir $(THIS_FILE))
THIS_FILE := $(lastword $(MAKEFILE_LIST))
PROJECT_NAME := $(notdir $(CURDIR))
#  ────────────────────────────────────────────────────────────────────
PKGBUILDS:=$(sort $(patsubst ./%/,%,$(dir $(call rwildcard,./,PKGBUILD)))) 
#  ────────────────────────────────────────────────────────────────────
.PHONY: all
all: help

# Add the following 'help' target to your Makefile
# And add help text after each target name starting with '\#\#'

.PHONY: help
help:			## Show this help
	@printf "$$(fgrep -h "##" $(MAKEFILE_LIST) | fgrep -v 'fgrep' | sed -e 's/\\$$//' | sed -e 's/\s.*##//')\n"

# 
#  ──── FORMAT ───────────────────────────────────────────────────────
# 
.PHONY: fmt
.SILENT: fmt
PKGBUILDS_FMT = $(PKGBUILDS:%=format-%)
fmt:			## Format files in repository
	$(info [INFO] Formatting PKGBUILDS)
ifeq (, $(shell ${WHICH} shfmt 2>${DEVNUL}))
	$(info 'shfmt' not found in path.)
	$(error aborting ...)
endif
	$(MAKE) --no-print-directory $(PKGBUILDS_FMT)

.PHONY: $(PKGBUILDS_FMT)
.SILENT: $(PKGBUILDS_FMT)
$(PKGBUILDS_FMT):	## Dynamically generated targets to format PKGBUILD and associated files
	$(eval name=$(@:format-%=%))
	$(eval command=shfmt -kp -i 2 -ci -w $(name)/PKGBUILD)
	$(command)
# 
#  ──── WORKSPACE CONTAINERS ─────────────────────────────────────────
# 
.PHONY: image
.SILENT: image
BUILD_PKGBUILDS_IMAGES = $(PKGBUILDS:%=%-image)
image:			## create docker images for building and submitting packages
	$(info [INFO] Creating workspace docker images)
ifeq (, $(shell ${WHICH} docker 2>${DEVNUL}))
	$(info 'docker' not found in path.)
	$(error aborting ...)
endif
	$(MAKE) --no-print-directory $(BUILD_PKGBUILDS_IMAGES)

.PHONY: $(BUILD_PKGBUILDS_IMAGES)
.SILENT: $(BUILD_PKGBUILDS_IMAGES)
$(BUILD_PKGBUILDS_IMAGES):%-image:format-%	## Dynamically generated targets (<name>-image) to build workspace docker images
	$(eval name=$(@:%-image=%))
	$(shell \
		DOCKER_BUILDKIT=1 \
		docker build \
		--secret id=id_rsa,src=$$HOME/.ssh/id_rsa \
		--build-arg PKG_NAME=$(name) \
    --build-arg USER=$(PROJECT_NAME) \
		--tag $(PROJECT_NAME)/$(name) . \
		)
# 
#  ──── SANDBOXES ─────────────────────────────────────────────────────
# 
.PHONY: sandbox
.SILENT: sandbox
PKGBUILDS_SANDBOXES = $(PKGBUILDS:%=%-sandbox)
sandbox:			## lists make targets for containerized pkgbuild sandboxes
	$(info [INFO] Containerized sandbox list : $(PKGBUILDS_SANDBOXES))

.PHONY: $(PKGBUILDS_SANDBOXES)
.SILENT: $(PKGBUILDS_SANDBOXES)
$(PKGBUILDS_SANDBOXES):%-sandbox:%-image	## Dynamically generated targets (<name>-sandbox) to create an container for specific pkgbuild
	$(eval name=$(@:%-sandbox=%))
	docker run \
		--volume "$(name):/workspace" \
		--volume "$(PWD)/$(name):/home/$(PROJECT_NAME)/$(name)" \
		--rm \
    --entrypoint "" \
		-it \
		"$(PROJECT_NAME)/${name}" /bin/sh -c \
    'ln -sf "$$HOME/$(name)/PKGBUILD" "/workspace/PKGBUILD" && bash'
