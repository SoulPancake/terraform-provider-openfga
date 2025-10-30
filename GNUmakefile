default: fmt lint generate

VERSION ?=
GOOS := $(shell go env GOOS)
GOARCH := $(shell go env GOARCH)
INSTALL_DIR := $(HOME)/.terraform.d/plugins/openfga/openfga/openfga/$(VERSION)/$(GOOS)_$(GOARCH)

build:
ifndef VERSION
	$(error VERSION is required. Usage: make build VERSION=0.1.0)
endif
	mkdir -p bin
	go build -ldflags "-X main.version=$(VERSION)" -o bin/terraform-provider-openfga_v$(VERSION)

install: build
	mkdir -p $(INSTALL_DIR)
	cp bin/terraform-provider-openfga_v$(VERSION) $(INSTALL_DIR)/terraform-provider-openfga_v$(VERSION)

clean:
	rm -rf $(HOME)/.terraform.d/plugins/openfga/openfga/openfga/

lint:
	golangci-lint run

generate:
	cd tools; go generate ./...

fmt:
	gofmt -s -w -e .

test:
	go test -v -cover -timeout=120s -parallel=10 ./...

testacc:
	TF_ACC=1 go test -v -cover -timeout 120m -p=1 ./...

.PHONY: fmt lint test testacc build install generate clean
