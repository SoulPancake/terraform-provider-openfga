# Testing Guide for Terraform Provider OpenFGA

This guide explains how to test the Terraform Provider for OpenFGA, both during development and before contributing changes.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Build Commands](#build-commands)
- [Testing Workflows](#testing-workflows)
- [Local Development Testing](#local-development-testing)
- [CI/CD Testing](#cicd-testing)

## Prerequisites

Before you begin testing, ensure you have the following installed:

- [Go](https://golang.org/doc/install) >= 1.22
- [Terraform](https://developer.hashicorp.com/terraform/downloads) >= 1.0
- [Docker](https://docs.docker.com/get-docker/) (for running OpenFGA locally)
- [golangci-lint](https://golangci-lint.run/usage/install/) (for linting)

## Build Commands

The provider includes a comprehensive Makefile with various targets to help with building, testing, and development.

### Available Make Targets

Run `make help` to see all available targets:

```bash
make help
```

### Build the Provider

To build the provider binary with a specific version:

```bash
make build VERSION=0.2.0
```

**What this does:**
- Compiles the provider source code into a binary
- Embeds the version information using Go's ldflags
- Places the binary in the `./out` directory with the naming convention: `terraform-provider-openfga_v{VERSION}`

**Difference from `go build`:**
- `go build` creates a binary without version information and doesn't follow Terraform's naming conventions
- `make build` properly versions the binary and prepares it for installation

### Install the Provider Locally

To build and install the provider to your local Terraform plugins directory:

```bash
make install VERSION=0.2.0
```

**What this does:**
- Runs the `build` target first
- Creates the necessary directory structure in `~/.terraform.d/plugins/`
- Installs the binary at: `~/.terraform.d/plugins/registry.terraform.io/openfga/openfga/{VERSION}/{OS}_{ARCH}/terraform-provider-openfga_v{VERSION}`

**Difference from `go install`:**
- `go install` places the binary in `$GOPATH/bin` which isn't where Terraform looks for custom providers
- `make install` places the binary in the correct Terraform plugin directory structure, making it immediately available for local testing

### Clean Up

To remove all locally installed provider binaries and build artifacts:

```bash
make clean
```

This removes:
- All versions from `~/.terraform.d/plugins/registry.terraform.io/openfga/openfga/`
- The `./out` build directory

## Testing Workflows

### 1. Unit Tests

Unit tests run quickly and don't require external dependencies:

```bash
make test
```

This runs all unit tests with:
- Verbose output (`-v`)
- Coverage reporting (`-cover`)
- 120-second timeout
- Parallel execution across 10 goroutines

### 2. Acceptance Tests

Acceptance tests verify the provider works correctly with a real OpenFGA instance. These tests:
- Create actual resources
- Make real API calls
- Take significantly longer to run

#### Start OpenFGA Locally

Before running acceptance tests, start a local OpenFGA instance:

```bash
docker compose -f docker/docker-compose.yaml up
```

This starts OpenFGA with default configuration accessible at `http://localhost:8080`.

#### Run Acceptance Tests

```bash
make testacc
```

Or explicitly set the environment variable:

```bash
TF_ACC=1 make testacc
```

**Note:** Acceptance tests are skipped by default unless `TF_ACC=1` is set to prevent accidental execution.

### 3. Linting

Check code quality and style:

```bash
make lint
```

This runs `golangci-lint` with the configuration defined in `.golangci.yml`.

### 4. Code Formatting

Format all Go code according to standards:

```bash
make fmt
```

This runs `gofmt` with the `-s` (simplify) flag.

### 5. Generate Documentation

Update provider documentation:

```bash
make generate
```

This generates documentation from the provider schema and examples.

## Local Development Testing

### End-to-End Testing Workflow

Here's a complete workflow for testing your changes locally:

1. **Format and Lint Your Code**
   ```bash
   make fmt
   make lint
   ```

2. **Run Unit Tests**
   ```bash
   make test
   ```

3. **Build and Install Your Changes**
   ```bash
   make install VERSION=0.0.1-dev
   ```

4. **Create a Test Terraform Configuration**
   
   Create a file `test.tf`:
   ```terraform
   terraform {
     required_providers {
       openfga = {
         source  = "registry.terraform.io/openfga/openfga"
         version = "0.0.1-dev"
       }
     }
   }

   provider "openfga" {
     api_url = "http://localhost:8080"
   }

   resource "openfga_store" "test" {
     name = "Test Store"
   }
   ```

5. **Initialize Terraform**
   ```bash
   terraform init
   ```

6. **Test Your Provider**
   ```bash
   terraform plan
   terraform apply
   ```

7. **Clean Up After Testing**
   ```bash
   terraform destroy
   make clean
   ```

### Testing Specific Features

When adding or modifying a specific feature:

1. Write unit tests for your changes in the appropriate `*_test.go` file
2. Add acceptance tests if the feature involves resources or data sources
3. Update documentation by adding examples in the appropriate directory
4. Run `make generate` to update generated docs
5. Test the feature manually using the workflow above

### Debugging

To run the provider in debug mode:

```bash
go run main.go -debug
```

Then set the `TF_REATTACH_PROVIDERS` environment variable as instructed in the output and run Terraform commands in another terminal.

## CI/CD Testing

The repository includes GitHub Actions workflows that automatically:

- Run unit tests on every push
- Run acceptance tests on pull requests (when safe)
- Lint code
- Build the provider

Check `.github/workflows/` for the exact CI/CD configuration.

### Before Submitting a Pull Request

Run the complete test suite:

```bash
make fmt
make lint
make test
docker compose -f docker/docker-compose.yaml up -d
make testacc
docker compose -f docker/docker-compose.yaml down
```

## Common Issues

### Issue: Provider Not Found After Installation

**Solution:** Ensure you're using the correct version in your Terraform configuration that matches the VERSION you built with.

### Issue: Acceptance Tests Fail

**Solution:** 
- Verify OpenFGA is running: `curl http://localhost:8080/healthz`
- Check logs: `docker compose -f docker/docker-compose.yaml logs`

### Issue: golangci-lint Not Found

**Solution:** Install golangci-lint:
```bash
go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest
```

## Additional Resources

- [Terraform Plugin Development](https://developer.hashicorp.com/terraform/plugin)
- [OpenFGA Documentation](https://openfga.dev/docs)
- [Contributing Guidelines](CONTRIBUTING.md)
- [Repository README](README.md)

## Questions?

If you have questions or run into issues:
1. Check existing [GitHub Issues](https://github.com/openfga/terraform-provider-openfga/issues)
2. Join the [OpenFGA Community](https://openfga.dev/community)
3. Create a new issue with details about your problem
