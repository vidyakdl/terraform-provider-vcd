TEST?=$$(go list ./... )
GOFMT_FILES?=$$(find . -name '*.go' )
WEBSITE_REPO=github.com/hashicorp/terraform-website
GIT_DESCRIBE=$(shell git describe --tags)
PKG_NAME=vcd

default: build

# builds the plugin injecting output of `git describe` to BuildVersion variable
build: fmtcheck
	go install -ldflags="-X 'github.com/vmware/terraform-provider-vcd/v4/vcd.BuildVersion=$(GIT_DESCRIBE)'"

# builds the plugin with race detector enabled and injecting output of `git describe` to BuildVersion variable
buildrace: fmtcheck
	go install --race -ldflags="-X 'github.com/vmware/terraform-provider-vcd/v4/vcd.BuildVersion=$(GIT_DESCRIBE)'"

# creates a .zip archive of the code
dist:
	git archive --format=zip -o source.zip HEAD
	git archive --format=tar HEAD | gzip -c > source.tar.gz

# triggers cleanup by executing a non existent test
cleanup:
	cd vcd && go test -tags ALL -run "NoSuchTest\b"  -v -timeout 0
	

# builds and deploys the plugin
install: build
	@sh -c "'$(CURDIR)/scripts/install-plugin.sh'"

# builds and deploys the plugin with race detector enabled (useful for troubleshooting)
installrace: buildrace
	@sh -c "'$(CURDIR)/scripts/install-plugin.sh'"

# makes .tf files from test templates
test-binary-prepare: install
	@sh -c "'$(CURDIR)/scripts/runtest.sh' short-provider"
	@sh -c "'$(CURDIR)/scripts/runtest.sh' binary-prepare"


# validates HCL files without executing them
test-binary-validate: install
	@sh -c "'$(CURDIR)/scripts/runtest.sh' short-provider"
	@sh -c "'$(CURDIR)/scripts/runtest.sh' binary-validate"

# runs test using Terraform binary as Org user
test-binary-orguser: install
	@sh -c "'$(CURDIR)/scripts/runtest.sh' short-provider-orguser"
	@sh -c "'$(CURDIR)/scripts/runtest.sh' binary"

# runs upgrade test using Terraform binary
test-upgrade:
	@sh -c "'$(CURDIR)/scripts/test-upgrade.sh'"

# makes .tf files from test templates for upgrade testing, but does not execute them
test-upgrade-prepare:
	@sh -c "skip_upgrade_execution=1 '$(CURDIR)/scripts/test-upgrade.sh'"

# runs test using Terraform binary as system administrator using binary with race detection enabled
test-binary: installrace
	@sh -c "'$(CURDIR)/scripts/runtest.sh' short-provider"
	@sh -c "'$(CURDIR)/scripts/runtest.sh' binary"

# Retrieves the authorization token
token:
	@sh -c "'$(CURDIR)/scripts/runtest.sh' token"

# runs staticcheck
static: fmtcheck
	@sh -c "'$(CURDIR)/scripts/runtest.sh' static"

# security runs the source code security analysis tool `gosec`
security: fmtcheck
	@./scripts/gosec.sh

# runs the unit tests
testunit: fmtcheck
	@sh -c "'$(CURDIR)/scripts/runtest.sh' unit"

# Runs the basic execution test
test: testunit tagverify
	@sh -c "'$(CURDIR)/scripts/runtest.sh' short"

# Runs the full acceptance test as Org user
testacc-orguser: testunit
	@sh -c "'$(CURDIR)/scripts/runtest.sh' acceptance-orguser"

# Runs the full acceptance test as system administrator
testacc: testunit
	@sh -c "'$(CURDIR)/scripts/runtest.sh' acceptance"

# Runs the full acceptance test as system administrator
testacc-coverage: testunit
	@sh -c "'$(CURDIR)/scripts/runtest.sh' acceptance-coverage"
	
# Runs the acceptance test for tm
testtm-acc: fmtcheck testunit
	@sh -c "'$(CURDIR)/scripts/runtest.sh' tm-acceptance"

# Runs the acceptance test for tm with coverage
testtm-acc-coverage: fmtcheck
	@sh -c "'$(CURDIR)/scripts/runtest.sh' tm-coverage"

# Runs the acceptance test as system administrator for search label
test-search: testunit
	@sh -c "'$(CURDIR)/scripts/runtest.sh' search"

# Runs full acceptance test sequentially (using "-parallel 1" flag for go test)
testacc-race-seq: testunit
	@sh -c "'$(CURDIR)/scripts/runtest.sh' sequential-acceptance"

# Runs the acceptance test with tag 'multiple'
testmulti: fmtcheck
	@sh -c "'$(CURDIR)/scripts/runtest.sh' multiple"

# Runs the acceptance test for org
testorg: fmtcheck
	@sh -c "'$(CURDIR)/scripts/runtest.sh' org"

# Runs the acceptance test for catalog
testcatalog: fmtcheck
	@sh -c "'$(CURDIR)/scripts/runtest.sh' catalog"

# Runs the acceptance test for vapp
testvapp: fmtcheck
	@sh -c "'$(CURDIR)/scripts/runtest.sh' vapp"

# Runs the acceptance test for lb
testlb: fmtcheck
	@sh -c "'$(CURDIR)/scripts/runtest.sh' lb"

# Runs the acceptance test for user
testuser: fmtcheck
	@sh -c "'$(CURDIR)/scripts/runtest.sh' user"

# Runs the acceptance test for vm
testvm: fmtcheck
	@sh -c "'$(CURDIR)/scripts/runtest.sh' vm"

# Runs the acceptance test for gateway
testgateway: fmtcheck
	@sh -c "'$(CURDIR)/scripts/runtest.sh' gateway"

# Runs the acceptance test for network
testnetwork: fmtcheck
	@sh -c "'$(CURDIR)/scripts/runtest.sh' network"

# Runs the acceptance test for external network
testextnetwork: fmtcheck
	@sh -c "'$(CURDIR)/scripts/runtest.sh' extnetwork"


# runs Tenant Manager test using Terraform binary as system administrator using binary with race detection enabled
testtm-binary: installrace
	@sh -c "'$(CURDIR)/scripts/runtest.sh' short-provider-tm"
	@sh -c "'$(CURDIR)/scripts/runtest.sh' binary"

# generates  Tenant Manager testing scripts in 'vcd/test-artifacts'test using Terraform binary as system administrator
testtm-binary-prepare: install
	@sh -c "'$(CURDIR)/scripts/runtest.sh' short-provider-tm"
	@sh -c "'$(CURDIR)/scripts/runtest.sh' binary-prepare"

# vets all .go files
vet:
	@echo "go vet ."
	@go vet -tags ALL $$(go list ./... ) ; if [ $$? -ne 0 ]; then \
		echo ""; \
		echo "Vet found suspicious constructs. Please check the reported constructs"; \
		echo "and fix them if necessary before submitting the code for review."; \
		exit 1; \
	fi

# formats all .go files
fmt:
	gofmt -s -w $(GOFMT_FILES)

# runs a Go format check
fmtcheck:
	@sh -c "'$(CURDIR)/scripts/gofmtcheck.sh'"

# runs HCL validation
hclcheck:
	@sh -c "'$(CURDIR)/scripts/hcl-check.sh'"

# runs the go tidy directory check
tidy-check:
	go mod tidy
	git diff --exit-code

# checks that the code can compile
test-compile:
	cd vcd && go test -race -tags ALL -c .

# checks that tagged tests can run independently
tagverify:
	@scripts/test-tags.sh

# builds the website and allows running it from localhost
website:
ifeq (,$(wildcard $(GOPATH)/src/$(WEBSITE_REPO)))
	echo "$(WEBSITE_REPO) not found in your GOPATH (necessary for layouts and assets), get-ting..."
	git clone https://$(WEBSITE_REPO) $(GOPATH)/src/$(WEBSITE_REPO)
endif
	@$(MAKE) -C $(GOPATH)/src/$(WEBSITE_REPO) website-provider PROVIDER_PATH=$(shell pwd) PROVIDER_NAME=$(PKG_NAME)

# tests the website files for broken link
website-test:
ifeq (,$(wildcard $(GOPATH)/src/$(WEBSITE_REPO)))
	echo "$(WEBSITE_REPO) not found in your GOPATH (necessary for layouts and assets), get-ting..."
	git clone https://$(WEBSITE_REPO) $(GOPATH)/src/$(WEBSITE_REPO)
endif
	@$(MAKE) -C $(GOPATH)/src/$(WEBSITE_REPO) website-provider-test PROVIDER_PATH=$(shell pwd) PROVIDER_NAME=$(PKG_NAME)

.PHONY: build test testacc-race-seq testacc vet static fmt fmtcheck tidy-check test-compile website website-test

