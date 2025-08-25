# Module Resolution Troubleshooting Guide

This guide helps diagnose and fix Go module resolution issues in the 1Password Secrets Action project.

## Overview

The 1Password Secrets Action is a Go module with the following key characteristics:

- **Module Name**: `github.com/ModeSevenIndustrialSolutions/1password-secrets-action`
- **Repository**: `https://github.com/ModeSevenIndustrialSolutions/1password-secrets-action`
- **Go Version**: 1.23
- **Main Package**: `./cmd/op-secrets-action`

## Common Issues

### 1. "no required module provides package" Errors

**Symptoms:**
```
no required module provides package github.com/ModeSevenIndustrialSolutions/1password-secrets-action/internal/config; to add it:
```

**Root Causes:**
- Module name mismatch between `go.mod` and repository URL
- Working directory issues in CI/CD environments
- Go module cache corruption
- Missing or corrupted `go.mod` file

**Solutions:**

#### Check Module Identity
```bash
# Verify module name matches repository
go list -m
# Should output: github.com/ModeSevenIndustrialSolutions/1password-secrets-action

# Check module directory
go list -m -f '{{.Dir}}'
# Should point to current project directory
```

#### Fix Module Issues
```bash
# Clean module cache
go clean -modcache

# Re-download dependencies
go mod download
go mod verify

# Ensure module is tidy
go mod tidy
```

### 2. Import Path Resolution Failures

**Symptoms:**
```
package github.com/ModeSevenIndustrialSolutions/1password-secrets-action/internal/app is not in GOROOT
```

**Solutions:**

#### Verify Package Structure
```bash
# List all packages in module
go list ./...

# List internal packages specifically
go list ./internal/...

# Check if packages can be imported
go list ./cmd/op-secrets-action
```

#### Validate Internal Packages
```bash
# Ensure internal directory exists
ls -la internal/

# Check for Go files in internal packages
find internal -name "*.go" -type f | head -10
```

### 3. GitHub Actions Environment Issues

**Symptoms:**
- Works locally but fails in GitHub Actions
- Different behavior between CI runners

**Solutions:**

#### Debug GitHub Actions Environment
Add this step to your workflow:
```yaml
- name: "Debug Module Environment"
  run: |
    echo "PWD: $(pwd)"
    echo "GITHUB_WORKSPACE: $GITHUB_WORKSPACE"
    echo "Go version: $(go version)"
    echo "Module: $(go list -m)"
    echo "Module dir: $(go list -m -f '{{.Dir}}')"
    ls -la
    go mod download
    go mod verify
    go list ./...
```

#### Fix Working Directory Issues
Ensure your workflow steps run in the correct directory:
```yaml
- name: "Build"
  working-directory: ${{ github.workspace }}
  run: |
    go build -v ./cmd/op-secrets-action
```

## Automated Diagnostics

### Using the Test Script

Run the comprehensive module resolution test:

```bash
# Make script executable
chmod +x scripts/test-module.sh

# Run diagnostics
./scripts/test-module.sh
```

This script will:
- ✅ Verify module identity and structure
- ✅ Test package resolution
- ✅ Validate build process
- ✅ Check import paths
- ✅ Verify repository alignment

### Using Makefile Targets

```bash
# Debug module setup
make debug-mod

# Validate module configuration
make mod-validate

# Test CI build process
make ci-build

# Run comprehensive checks
make check
```

## Environment-Specific Fixes

### Local Development

```bash
# Set up development environment
make dev-setup

# Quick validation
make dev-test

# Full validation
make check
```

### GitHub Actions

#### Add Module Debugging to Workflow
```yaml
- name: "Setup Go"
  uses: actions/setup-go@v5
  with:
    go-version-file: 'go.mod'
    cache: true

- name: "Debug Module Setup"
  run: |
    go version
    go list -m
    go mod download
    go mod verify
    go list ./internal/...

- name: "Build"
  run: |
    go mod tidy
    go build -v ./cmd/op-secrets-action
```

#### Use Makefile in CI
```yaml
- name: "CI Build"
  run: make ci-build
```

### Docker/Container Environments

```bash
# Build container image
make docker-build

# Test in container
make docker-test
```

## Manual Validation Steps

### 1. Basic Module Check
```bash
cd /path/to/project
go mod verify
go list -m
```

### 2. Package Resolution Check
```bash
go list ./...
go list ./internal/...
go list ./cmd/op-secrets-action
```

### 3. Build Test
```bash
go build -v ./cmd/op-secrets-action
./op-secrets-action version
```

### 4. Import Validation
```bash
# Test specific imports
go list github.com/ModeSevenIndustrialSolutions/1password-secrets-action/internal/config
go list github.com/ModeSevenIndustrialSolutions/1password-secrets-action/internal/app
```

## Advanced Troubleshooting

### Go Module Debug Information
```bash
# Show module graph
go mod graph

# Show module dependencies
go list -m all

# Show module requirements
go list -m -json

# Environment information
go env
```

### Cache and Proxy Issues
```bash
# Check module cache location
go env GOMODCACHE

# Check proxy settings
go env GOPROXY

# Disable proxy temporarily
GOPROXY=direct go mod download

# Clear module cache
go clean -modcache
```

### Workspace Issues
```bash
# Check for workspace files
find . -name "go.work*" -o -name "*.work"

# If workspace files exist, they may interfere
# Consider removing or configuring properly
```

## Prevention Strategies

### 1. Consistent Development Setup
- Use `make dev-setup` for initial setup
- Run `make check` before commits
- Use `make pre-commit` as a git hook

### 2. CI/CD Best Practices
- Always run module validation in CI
- Use caching appropriately
- Add debugging steps for troubleshooting

### 3. Regular Maintenance
```bash
# Keep dependencies updated
make deps-update

# Regular module validation
make mod-verify

# Clean rebuilds
make clean && make build
```

## Getting Help

If issues persist after following this guide:

1. **Run Full Diagnostics**: `./scripts/test-module.sh`
2. **Check CI Logs**: Look for the debug output in failed GitHub Actions
3. **Compare Environments**: Ensure local and CI environments match
4. **Review Recent Changes**: Check if recent commits introduced the issue

### Debug Information to Collect

When reporting issues, include:

```bash
# Environment information
go version
go env
pwd
ls -la

# Module information
go list -m
go list -m -f '{{.Dir}}'
go mod verify

# Package information
go list ./...
```

## Quick Reference

| Command | Purpose |
|---------|---------|
| `make debug-mod` | Show module debug information |
| `make mod-validate` | Validate module setup |
| `make ci-build` | Test CI build process |
| `./scripts/test-module.sh` | Comprehensive diagnostics |
| `make clean && make build` | Clean rebuild |
| `go clean -modcache` | Clear module cache |
| `go mod tidy` | Clean up module files |