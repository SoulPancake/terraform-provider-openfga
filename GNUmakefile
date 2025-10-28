# Variables
HOSTNAME = registry.terraform.io
NAMESPACE = openfga
NAME = openfga
BINARY = terraform-provider-${NAME}

BUILD_DIR ?= $(CURDIR)/out

GO_OS ?= $(shell go env GOOS)
GO_ARCH ?= $(shell go env GOARCH)
GO_BIN ?= $(shell go env GOPATH)/bin

# Colors for the printf
RESET = $(shell tput sgr0)
COLOR_WHITE = $(shell tput setaf 7)
COLOR_BLUE = $(shell tput setaf 4)
COLOR_YELLOW = $(shell tput setaf 3)
TEXT_INVERSE = $(shell tput smso)

# Default target
default: fmt lint install generate

# Build commands
build: ## Build the provider binary. Usage: "make build VERSION=0.2.0"
	@if [ -z "$(VERSION)" ]; \
	then \
	  echo "Please provide a version. Example: make build VERSION=0.2.0" && exit 1; \
	fi
	@echo "${COLOR_BLUE}Building the provider binary${RESET}"
	@mkdir -p "${BUILD_DIR}"
	@go build -v -ldflags "-X main.version=${VERSION}" -o "${BUILD_DIR}/${BINARY}_v$(VERSION)"
	@echo "${COLOR_BLUE}Binary built successfully at: ${BUILD_DIR}/${BINARY}_v$(VERSION)${RESET}"

install: build ## Install the provider as a terraform plugin. Usage: "make install VERSION=0.2.0"
	@echo "${COLOR_BLUE}Installing the provider to Terraform plugins directory${RESET}"
	@mkdir -p "${HOME}/.terraform.d/plugins/${HOSTNAME}/${NAMESPACE}/${NAME}/$(VERSION)/${GO_OS}_${GO_ARCH}"
	@mv "${BUILD_DIR}/${BINARY}_v$(VERSION)" "${HOME}/.terraform.d/plugins/${HOSTNAME}/${NAMESPACE}/${NAME}/$(VERSION)/${GO_OS}_${GO_ARCH}/${BINARY}_v$(VERSION)"
	@echo "${COLOR_BLUE}Provider installed successfully at: ${HOME}/.terraform.d/plugins/${HOSTNAME}/${NAMESPACE}/${NAME}/$(VERSION)/${GO_OS}_${GO_ARCH}/${RESET}"

clean: ## Clean up locally installed provider binaries
	@echo "${COLOR_YELLOW}Cleaning locally installed provider binaries${RESET}"
	@rm -rf "${HOME}/.terraform.d/plugins/${HOSTNAME}/${NAMESPACE}/${NAME}"
	@rm -rf "${BUILD_DIR}"
	@echo "${COLOR_YELLOW}Cleanup complete${RESET}"

# Development commands
lint: ## Run golangci-lint
	@echo "${COLOR_BLUE}Running golangci-lint${RESET}"
	@command -v golangci-lint >/dev/null 2>&1 || { \
		echo "${COLOR_YELLOW}golangci-lint not found. Please install it:${RESET}"; \
		echo "  go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest"; \
		echo "Or visit: https://golangci-lint.run/usage/install/"; \
		exit 1; \
	}
	@golangci-lint run

generate: ## Generate documentation
	@echo "${COLOR_BLUE}Generating documentation${RESET}"
	@cd tools; go generate ./...

fmt: ## Format Go code
	@echo "${COLOR_BLUE}Formatting Go code${RESET}"
	@gofmt -s -w -e .

# Testing commands
test: ## Run unit tests
	@echo "${COLOR_BLUE}Running unit tests${RESET}"
	@go test -v -cover -timeout=120s -parallel=10 ./...

testacc: ## Run acceptance tests (requires TF_ACC=1)
	@echo "${COLOR_BLUE}Running acceptance tests${RESET}"
	@TF_ACC=1 go test -v -cover -timeout 120m -p=1 ./...

# Help command
help: ## Show this help message
	@echo "${COLOR_BLUE}Available targets:${RESET}"
	@awk 'BEGIN {FS = ":.*##"; printf "\n"} /^[a-zA-Z_-]+:.*?##/ { printf "  ${COLOR_WHITE}%-15s${RESET} %s\n", $$1, $$2 } /^##@/ { printf "\n${COLOR_YELLOW}%s${RESET}\n", substr($$0, 5) } ' $(MAKEFILE_LIST)

.PHONY: help fmt lint test testacc build install generate clean
