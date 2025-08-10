# MkDocs Container Builder

**Enterprise-Grade Documentation Container System for 40docs Platform**

A specialized Docker container build system that provides secure, optimized MkDocs documentation containers with Material theme integration and comprehensive plugin ecosystem. Part of the 40docs platform's Documentation-as-Code infrastructure.

## 🏗️ Architecture Overview

This repository builds production-ready MkDocs containers with three distinct architectural approaches:

- **Standard Build** (`Dockerfile`): Full-featured Python 3.11 container with comprehensive tooling
- **Security-Optimized** (`Dockerfile.optimized`): Multi-stage distroless build with minimal attack surface
- **Alpine Lightweight** (`Dockerfile.alpine`): Compact Alpine-based build extending squidfunk/mkdocs-material

### Key Features

✅ **Material Theme Integration**: Built-in mkdocs-material with theme linking and customization  
✅ **Comprehensive Plugin Ecosystem**: 28+ MkDocs plugins including PDF export, git integration, and content enhancement  
✅ **Playwright Support**: Browser automation for PDF generation and visual testing  
✅ **Multi-Language Font Support**: Noto, DejaVu, and Asian language fonts for international documentation  
✅ **Security Hardening**: Non-root execution, distroless images, and security scanning integration  
✅ **GitOps Ready**: Seamless integration with 40docs platform's Flux-based deployment system  

## 🚀 Quick Start

### Building Containers

```bash
# Clone repository and dependencies
git clone <repository-url>
cd mkdocs

# Standard development build
docker build -f Dockerfile -t mkdocs-standard .

# Security-optimized production build  
docker build -f Dockerfile.optimized -t mkdocs-secure .

# Lightweight Alpine build
docker build -f Dockerfile.alpine -t mkdocs-alpine .
```

### Running Documentation Server

```bash
# Run development server (standard container)
docker run -p 8000:8000 -v $(pwd):/tmp/docs mkdocs-standard

# Run with custom configuration
docker run -p 8000:8000 \
  -v $(pwd):/tmp/docs \
  -v $(pwd)/mkdocs.yml:/tmp/mkdocs.yml \
  mkdocs-standard

# Interactive debugging shell
docker run -it --entrypoint /bin/bash mkdocs-standard
```

## 📦 Plugin Ecosystem

This container includes 28+ specialized MkDocs plugins organized by functionality:

### Content Enhancement
- `mkdocs-macros-plugin` - Template macros and variables
- `mkdocs-include-markdown-plugin` - Include external markdown files
- `mkdocs-content-tabs` - Tabbed content sections
- `mkdocs-badges` - GitHub-style badges

### Git Integration & Versioning
- `mkdocs-git-authors-plugin` - Author attribution from git history
- `mkdocs-git-committers-plugin-2` - Contributor tracking
- `mkdocs-git-revision-date-localized-plugin` - Localized modification dates

### Visual & Media
- `mkdocs-glightbox` - Image lightbox galleries
- `mkdocs-drawio` - Draw.io diagram integration
- `lightgallery` - Advanced image galleries
- `mkdocs-asciinema-player` - Terminal recording playback

### PDF & Export
- `mkdocs-pdf-export-plugin` - PDF export functionality
- `mkdocs-with-pdf` - Advanced PDF generation with WeasyPrint
- `mkdocs-exporter` - Multi-format export capabilities

### Navigation & Structure
- `mkdocs-awesome-pages-plugin` - Advanced page organization
- `mkdocs-literate-nav` - Literate navigation structure
- `mkdocs-section-index` - Section index pages
- `mkdocs-enumerate-headings-plugin` - Automatic heading numbering

### Code Documentation
- `mkdocstrings[crystal,python]` - API documentation from code
- `pymdown-extensions` - Python Markdown extensions
- `pygments` - Syntax highlighting

### Performance & Optimization
- `mkdocs-minify-plugin` - Asset minification
- `mkdocs-monorepo-plugin` - Multi-repository documentation

## 🔐 Security Features

### Multi-Stage Security (Dockerfile.optimized)

The security-optimized build implements defense-in-depth:

- **Distroless Base Image**: `gcr.io/distroless/python3-debian11` for minimal attack surface
- **Non-Root Execution**: Dedicated `mkdocs` user (uid:1000, gid:1000)
- **Multi-Stage Build**: Separates build tools from runtime environment
- **Version Pinning**: Fixed versions for pip, setuptools, wheel, and critical dependencies
- **Security Metadata**: OpenContainers labels for security scanning integration
- **Health Checks**: Built-in health monitoring for production deployments

### Security Standards Compliance

- Container image vulnerability scanning ready
- OWASP container security best practices
- Minimal runtime dependencies
- No secrets or credentials in images
- Secure file permissions (600/700 for sensitive files)

## 🔧 CI/CD Integration

### GitHub Actions Workflow

The repository includes automated CI/CD via `.github/workflows/container.yml`:

```yaml
# Triggered on push to main or manual dispatch
# Permissions: packages:write, contents:read
# Concurrency control to prevent duplicate builds
```

**Workflow Steps**:
1. **Dependency Integration**: Clones squidfunk/mkdocs-material for latest Material theme
2. **Multi-Architecture Build**: Docker Buildx for cross-platform support  
3. **Registry Publishing**: Pushes to GitHub Container Registry (`ghcr.io`)
4. **Security Scanning**: Integrated with GitHub's security features

### Container Registry

Images are published to GitHub Container Registry:
- **Registry**: `ghcr.io/40docs/mkdocs:latest`
- **Architecture**: Multi-platform (amd64, arm64)
- **Tagging**: Latest tag with commit SHA for traceability

## 🌍 40docs Platform Integration

### Theme Inheritance System

The container supports the 40docs theme inheritance model:

```dockerfile
# Default configuration inherits from theme repository
RUN echo "INHERIT: docs/theme/mkdocs.yml" > "/tmp/mkdocs.yml"
```

### GitOps Workflow Integration

- **Flux v2 Compatibility**: Seamless deployment to Kubernetes clusters
- **Multi-Repository Builds**: SSH key injection for secure cross-repo access
- **Content Mounting**: Documentation content from distributed repositories
- **Build Pipeline**: Integrated with platform's automated deployment system

### Cross-Repository Dependencies

The container expects:
- Theme configuration from `docs/theme/mkdocs.yml` 
- Content directories mounted at `/tmp/docs`
- Build output directory at `/tmp/site`
- Git repository context for plugin functionality

## 🐛 Troubleshooting

### Common Build Issues

```bash
# Font cache problems
docker run --rm mkdocs-standard fc-cache -f

# Git safe directory configuration  
docker run --rm mkdocs-standard git config --global --add safe.directory /tmp/docs

# Playwright browser installation
docker run --rm mkdocs-standard \
  env PLAYWRIGHT_BROWSERS_PATH=/ms-playwright playwright install chromium
```

### Container Debugging

```bash
# Inspect container layers and size
docker history mkdocs-standard

# Interactive debugging session
docker exec -it <container_id> /bin/bash

# Verify font installation
docker run --rm mkdocs-standard fc-list | grep -i noto

# Check plugin availability
docker run --rm mkdocs-standard pip list | grep mkdocs
```

### Performance Tuning

**Memory Usage by Build Type**:
- **Standard**: ~800MB-1GB (full ecosystem)
- **Optimized**: ~400-600MB (distroless)  
- **Alpine**: ~300-500MB (lightweight)

**Build Time Optimization**:
- Use Docker BuildKit: `DOCKER_BUILDKIT=1 docker build`
- Leverage layer caching for requirements.txt changes
- Consider multi-stage caching for faster rebuilds

## 🔄 Development Workflow

### Making Container Modifications

1. **Layer Optimization**: Maintain caching efficiency by placing frequently changing files last
2. **Version Pinning**: Pin versions for reproducible builds and security
3. **Multi-Variant Testing**: Test changes across all three Dockerfile variants
4. **Security Validation**: Update OpenContainers metadata and security labels
5. **Plugin Compatibility**: Verify new plugins work across all build types

### Contributing Guidelines

- Follow existing Dockerfile patterns for consistency
- Update requirements.txt with version constraints
- Test builds locally before submitting PRs
- Update CLAUDE.md documentation for architectural changes
- Validate security scanning results for optimized builds

## 📊 Performance Metrics

### Build Performance
- **Standard Build**: ~5-8 minutes (full compilation)
- **Optimized Build**: ~8-12 minutes (multi-stage)
- **Alpine Build**: ~3-5 minutes (lightweight)

### Runtime Performance
- **Startup Time**: <10 seconds for development server
- **Memory Footprint**: 300MB-1GB depending on build type
- **PDF Generation**: Playwright + WeasyPrint for high-quality output
- **Plugin Load Time**: <2 seconds for full plugin ecosystem

## 📚 Related Documentation

- [40docs Platform Overview](../README.md)
- [CLAUDE.md - Development Guidelines](./CLAUDE.md)
- [Infrastructure Documentation](../infrastructure/README.md)
- [Theme Repository](../theme/README.md)
- [MkDocs Material Documentation](https://squidfunk.github.io/mkdocs-material/)

## 🏷️ Version History

- **v1.0**: Initial multi-variant container system
- **Recent**: Security-optimized distroless builds
- **Latest**: Enhanced plugin ecosystem with 28+ plugins

---

**Maintained by**: 40docs Platform Team  
**License**: Part of 40docs platform ecosystem  
**Registry**: `ghcr.io/40docs/mkdocs:latest`