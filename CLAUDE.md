# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is the **mkdocs** component of the 40docs platform - a specialized Docker container build system for MkDocs documentation with Material theme. This directory contains multiple Dockerfile variants optimized for different use cases and security requirements.

## Container Architecture

### Available Dockerfile Variants

- **`Dockerfile`**: Standard build with full Python 3.11 and comprehensive plugin support
- **`Dockerfile.optimized`**: Multi-stage security-hardened build with distroless final image and non-root user
- **`Dockerfile.alpine`**: Lightweight Alpine-based build extending squidfunk/mkdocs-material

### Key Features

- **Material Theme Integration**: Built-in mkdocs-material with theme linking and customization
- **Comprehensive Plugin Ecosystem**: 40+ MkDocs plugins including PDF export, git integration, and content enhancement
- **Playwright Support**: Browser automation for PDF generation and testing
- **Font Support**: Multiple font families including Noto, DejaVu, and Asian language fonts
- **Multi-Architecture**: Supports various deployment scenarios from development to production

## Development Commands

### Container Build Commands

```bash
# Standard development build
docker build -f Dockerfile -t mkdocs-standard .

# Security-optimized production build
docker build -f Dockerfile.optimized -t mkdocs-secure .

# Lightweight Alpine build
docker build -f Dockerfile.alpine -t mkdocs-alpine .
```

### Local Development

```bash
# Run MkDocs development server (standard container)
docker run -p 8000:8000 -v $(pwd):/tmp/docs mkdocs-standard

# Run with custom config file
docker run -p 8000:8000 -v $(pwd):/tmp/docs -v $(pwd)/mkdocs.yml:/tmp/mkdocs.yml mkdocs-standard

# Interactive shell for debugging
docker run -it --entrypoint /bin/bash mkdocs-standard
```

## Security Considerations

### Optimized Container Security

The `Dockerfile.optimized` implements security best practices:
- **Multi-stage build**: Separates build dependencies from runtime
- **Distroless base**: Minimal attack surface with gcr.io/distroless/python3-debian11
- **Non-root user**: Runs as dedicated `mkdocs` user (uid:1000, gid:1000)
- **Version pinning**: Fixed versions for pip, setuptools, wheel, and key dependencies
- **Security labels**: OpenContainers standard metadata for security scanning

### Security Scanning Integration

The optimized build is designed for:
- Container image vulnerability scanning
- Security policy compliance
- Production deployment with minimal attack surface
- Health check implementation for monitoring

## Plugin Ecosystem

### Core Plugins Included

The `requirements.txt` includes comprehensive plugin support:

- **Content Enhancement**: mkdocs-macros-plugin, mkdocs-include-markdown-plugin
- **Git Integration**: mkdocs-git-authors-plugin, mkdocs-git-revision-date-localized-plugin
- **PDF Export**: mkdocs-pdf-export-plugin, mkdocs-with-pdf
- **Visual Elements**: mkdocs-glightbox, mkdocs-drawio, lightgallery
- **Navigation**: mkdocs-awesome-pages-plugin, mkdocs-literate-nav
- **Code Documentation**: mkdocstrings[crystal,python]
- **Performance**: mkdocs-minify-plugin

### Plugin Configuration

Plugins are configured through inheritance:
- Default config: `INHERIT: docs/theme/mkdocs.yml`
- Theme linking: Material theme templates linked to mkdocs and readthedocs directories
- Font configuration: Comprehensive font support for international content

## Build Optimization

### Layer Caching Strategy

The Dockerfiles are optimized for build caching:
1. **Dependencies First**: Copy requirements.txt and package.json before application code
2. **System Packages**: Install system dependencies in single RUN layers
3. **Python Packages**: Separate pip install commands for better caching
4. **Application Code**: Copy application files last for optimal rebuild performance

### Multi-Stage Benefits

The optimized build provides:
- **Reduced Image Size**: Final image contains only runtime dependencies
- **Security Hardening**: Build tools and development packages excluded from final image
- **Faster Deployment**: Smaller images deploy faster and use less storage

## Integration with 40docs Platform

### GitOps Integration

This container is used within the 40docs GitOps workflow:
- **Theme Inheritance**: Inherits styling from `theme/` repository
- **Content Mounting**: Documentation content mounted from various content repositories
- **Build Pipeline**: Integrated with Kubernetes deployment via Flux

### Cross-Repository Dependencies

The MkDocs container expects:
- Theme configuration from `docs/theme/mkdocs.yml`
- Content directories mounted at `/tmp/docs`
- Build output directory at `/tmp/site`

## Troubleshooting

### Common Build Issues

```bash
# Font cache issues
fc-cache -f

# Git safe directory issues
git config --global --add safe.directory /tmp/docs
git config --global --add safe.directory /tmp/site

# Playwright browser installation
PLAYWRIGHT_BROWSERS_PATH=/ms-playwright playwright install chromium
```

### Container Debugging

```bash
# Check container layers
docker history mkdocs-standard

# Inspect running container
docker exec -it <container_id> /bin/bash

# Check font availability
docker run --rm mkdocs-standard fc-list | grep -i noto
```

## Performance Considerations

### Memory Usage

- **Standard Build**: ~800MB-1GB for full plugin ecosystem
- **Optimized Build**: ~400-600MB with distroless base
- **Alpine Build**: ~300-500MB lightweight alternative

### Build Time Optimization

- Use Docker BuildKit for parallel builds
- Leverage multi-stage caching
- Consider using base image with pre-installed common dependencies

## Development Guidelines

### Container Modifications

When modifying Dockerfiles:
- Maintain layer optimization for caching
- Pin versions for reproducible builds
- Test all three variants for compatibility
- Update security labels and metadata
- Validate plugin compatibility

### Security Updates

Regular maintenance tasks:
- Update base image versions
- Pin security-critical dependencies
- Review and update plugin versions
- Scan containers for vulnerabilities
- Update OpenContainers metadata