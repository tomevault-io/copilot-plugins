## deployment-and-production

> - ✅ All tests passing (`npm test`)

# Deployment & Production Guidelines

## 🚀 Production Deployment

### Pre-deployment Checklist
- ✅ All tests passing (`npm test`)
- ✅ TypeScript compilation successful (`npm run build`)
- ✅ Code quality checks passing (`npm run lint`)
- ✅ 90%+ test coverage (`npm run test:coverage`)
- ✅ No security vulnerabilities (`npm audit`)
- ✅ Documentation updated
- ✅ Version bumped in [package.json](mdc:package.json)

### Build Process
The [scripts/build.sh](mdc:scripts/build.sh) handles production builds:
1. **Clean**: Remove previous build artifacts
2. **Type Check**: Validate TypeScript without compilation
3. **Compile**: TypeScript to JavaScript (ES2020/CommonJS)
4. **Package**: Copy essential files and set permissions
5. **Binaries**: Create CLI executables

### Distribution Channels

#### NPM Publishing
```bash
# Build and test
npm run build
npm test

# Publish to NPM
npm publish --access public

# Verify installation
npm install -g smartlead-cli
smartlead --version
```

#### GitHub Releases
Automated via [.github/workflows/ci.yml](mdc:.github/workflows/ci.yml):
- **Tags**: Create git tags for releases
- **Artifacts**: Build artifacts attached to releases
- **Changelog**: Auto-generated from commits
- **Cross-platform**: Builds for Windows, macOS, Linux

## 🔧 Installation Methods

### Global Installation (Recommended)
```bash
# From NPM (when published)
npm install -g smartlead-cli

# From source
git clone https://github.com/username/smartlead-cli.git
cd smartlead-cli
bash scripts/install.sh
```

### Development Installation
```bash
# Clone and build
git clone https://github.com/username/smartlead-cli.git
cd smartlead-cli
npm install
npm run build
npm run install-global
```

### Professional Installer
The [scripts/install.sh](mdc:scripts/install.sh) provides:
- **Environment Validation**: Node.js and npm version checks
- **Dependency Installation**: Automated npm install
- **Build Process**: TypeScript compilation
- **Global Installation**: CLI binary setup
- **Verification**: Installation testing and validation

## 🔐 Environment Configuration

### Production Environment Variables
Copy [environment.example](mdc:environment.example) to configure:

```bash
# Required
SMARTLEAD_API_KEY=your-production-api-key

# Optional Production Settings
NODE_ENV=production
MCP_SERVER_PORT=3001
RATE_LIMIT_REQUESTS=100
API_TIMEOUT=30000
```

### Configuration Files
- **Global Config**: `~/.smartlead-cli/config.json`
- **Module Configs**: `~/.smartlead-cli/{module}.json`
- **MCP Config**: [.smartlead-mcp.json](mdc:.smartlead-mcp.json)
- **Environment**: [environment.example](mdc:environment.example)

## 📊 Monitoring & Analytics

### Health Checks
```bash
# CLI Health
smartlead --version
smartlead show-config
smartlead modules

# API Connectivity
smartlead campaigns --limit 1
smartlead email-accounts --limit 1
```

### Performance Monitoring
- **Startup Time**: CLI initialization speed
- **Command Response**: API call performance
- **Memory Usage**: Resource consumption
- **Error Rates**: Command failure tracking

### Logging
- **Debug Mode**: `DEBUG=true` for detailed logs
- **Log Levels**: error, warn, info, debug
- **Error Tracking**: Comprehensive error logging
- **Performance Metrics**: Command timing and stats

## 🔄 CI/CD Pipeline

### GitHub Actions Workflow
The [.github/workflows/ci.yml](mdc:.github/workflows/ci.yml) includes:

#### Test Suite (`test` job)
- **Multi-Node**: Tests on Node.js 16, 18, 20
- **TypeScript**: Type checking and compilation
- **Code Quality**: ESLint and Prettier validation
- **Test Coverage**: Jest with coverage reporting
- **CLI Testing**: Command verification

#### Quality Assurance (`quality` job)
- **Strict TypeScript**: Enhanced type checking
- **Code Analysis**: ESLint with detailed reporting
- **Formatting**: Prettier consistency checks

#### Security Scanning (`security` job)
- **Dependency Audit**: npm audit for vulnerabilities
- **Security Analysis**: audit-ci for critical issues

#### Cross-Platform Builds (`build-matrix` job)
- **Multi-OS**: Ubuntu, Windows, macOS
- **Build Testing**: Compilation on all platforms
- **Installation**: Package and installation testing

#### NPM Publishing (`publish-npm` job)
- **Release Trigger**: Automated on GitHub releases
- **Production Build**: Full build and test cycle
- **NPM Deployment**: Automated publishing

### Release Process
1. **Version Bump**: Update version in [package.json](mdc:package.json)
2. **Changelog**: Update [docs/CHANGELOG.md](mdc:docs/CHANGELOG.md)
3. **Git Tag**: Create release tag
4. **GitHub Release**: Create release with notes
5. **Automated**: CI/CD handles NPM publishing

## 📦 Package Management

### Package Configuration
The [package.json](mdc:package.json) includes:
- **Files**: Only essential files in distribution
- **Binaries**: CLI command setup (`smartlead`, `sl`)
- **Dependencies**: Minimal production dependencies
- **Scripts**: Comprehensive build and development scripts

### Distribution Files
```
smartlead-cli/
├── dist/                    # Compiled JavaScript
├── docs/                    # Documentation
├── scripts/install.sh       # Installation script
├── LICENSE                  # MIT license
├── package.json            # Package configuration
└── README.txt              # Quick reference
```

## 🎯 Production Checklist

### Pre-Release
- [ ] All tests passing
- [ ] Documentation updated
- [ ] Version bumped
- [ ] Changelog updated
- [ ] Security audit clean
- [ ] Cross-platform testing
- [ ] Installation testing

### Release
- [ ] Git tag created
- [ ] GitHub release published
- [ ] NPM package published
- [ ] Installation verified
- [ ] Documentation deployed

### Post-Release
- [ ] Installation testing
- [ ] User feedback monitoring
- [ ] Performance tracking
- [ ] Issue tracking
- [ ] Next version planning

## 🚨 Troubleshooting

### Common Issues
- **Installation Failures**: Node.js version compatibility
- **Permission Errors**: Global installation permissions
- **API Connectivity**: Network or API key issues
- **Module Loading**: TypeScript compilation issues

### Debug Mode
```bash
# Enable debug logging
DEBUG=true smartlead config
NODE_ENV=development smartlead modules

# Verbose logging
CLI_LOG_LEVEL=debug smartlead campaigns
```

### Support Channels
- **GitHub Issues**: Bug reports and feature requests
- **Documentation**: Comprehensive guides in [docs/](mdc:docs/)
- **Community**: GitHub Discussions for support
- **Professional**: Email support for enterprise users

## 📈 Performance Optimization

### Build Optimization
- **TypeScript**: Optimized compilation settings
- **Bundle Size**: Minimal dependencies
- **Tree Shaking**: Dead code elimination
- **Caching**: Build artifact caching

### Runtime Performance
- **Lazy Loading**: Module loading on demand
- **Caching**: API response caching
- **Connection Pooling**: Efficient API connections
- **Memory Management**: Proper cleanup and disposal

---
> Source: [LeadMagic/cold-email-cli](https://github.com/LeadMagic/cold-email-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-09 -->
