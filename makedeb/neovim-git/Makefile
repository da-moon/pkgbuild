# vim: filetype=make syntax=make textwidth=0 softtabstop=0 tabstop=4 shiftwidth=4 fileencoding=utf-8 fileformat=unix smartindent autoindent noexpandtab
#  ────────────────────────────────────────────────────────────────────
.PHONY: $(shell egrep -o '^[a-zA-Z_-]+:' $(MAKEFILE_LIST) | sed 's/://')
.SILENT: $(shell egrep -o '^[a-zA-Z_-]+:' $(MAKEFILE_LIST) | sed 's/://')
.PHONY: all
all: help
#  ────────────────────────────────────────────────────────────────────
help: ## Show this help
	@printf "$$(fgrep -h "##" $(MAKEFILE_LIST) | fgrep -v 'fgrep' | sed -e 's/\\$$//' | sed -e 's/\s.*##//')\n"

clean:
	rm -rf neovim src pkg *.deb *.tar.gz

srcinfo: ## Generate srcinfo
	makedeb --printsrcinfo > .SRCINFO

--apt-update:
	sudo apt-get update

makedeb: ## Build the package while syncing dependencies
	makedeb -s --no-confirm

install: ## Installs the package
	makedeb --install --no-confirm

build: --apt-update srcinfo makedeb ## Main target for building a fresh package