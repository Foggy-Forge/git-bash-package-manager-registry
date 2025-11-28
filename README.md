# git-bash-package-manager-registry

Official package registry for [gbpm](https://github.com/Foggy-Forge/git-bash-package-manager) – the Git Bash Package Manager.

This repository contains YAML manifests that describe how to install CLI tools and utilities for Git Bash on Windows.

---

## Structure

```text
git-bash-package-manager-registry/
  README.md
  docs/
    manifest-spec.md
  packages/
    fzf/
      fzf.yaml
    bat/
      bat.yaml
    ripgrep/
      ripgrep.yaml
    jq/
      jq.yaml
    fd/
      fd.yaml
```

Each package lives in `packages/<name>/<name>.yaml`.

---

## Available Packages

| Package | Description | Homepage |
|---------|-------------|----------|
| **fzf** | General-purpose command-line fuzzy finder | [junegunn/fzf](https://github.com/junegunn/fzf) |
| **bat** | A cat(1) clone with syntax highlighting | [sharkdp/bat](https://github.com/sharkdp/bat) |
| **ripgrep** | Fast grep alternative (rg) | [BurntSushi/ripgrep](https://github.com/BurntSushi/ripgrep) |
| **jq** | Lightweight JSON processor | [jqlang/jq](https://github.com/jqlang/jq) |
| **fd** | Fast alternative to find | [sharkdp/fd](https://github.com/sharkdp/fd) |

---

## Using This Registry

Once you have `gbpm` installed:

```bash
# Update the registry (clone or pull latest)
gbpm update

# Install a package
gbpm install fzf

# List installed packages
gbpm list

# Uninstall a package
gbpm uninstall fzf
```

---

## Contributing

Want to add a package? Follow these steps:

1. **Fork this repository**

2. **Create a new package directory**:
   ```bash
   mkdir -p packages/yourpackage
   ```

3. **Create a manifest** (`packages/yourpackage/yourpackage.yaml`):
   - Follow the [manifest specification](./docs/manifest-spec.md)
   - Test it with `gbpm install --file packages/yourpackage/yourpackage.yaml`

4. **Submit a pull request**

### Guidelines

- Package names should be lowercase, alphanumeric (hyphens allowed)
- Manifests must include `name`, `version`, `platforms`, and `install` steps
- Only include packages with permissive licenses (MIT, Apache, BSD, etc.)
- Prefer official release binaries from trusted sources
- Include checksums when possible (will be required in future versions)

---

## Manifest Specification

See [docs/manifest-spec.md](./docs/manifest-spec.md) for complete details.

Quick example:

```yaml
name: fzf
version: 0.46.1
description: "A general-purpose command-line fuzzy finder"
homepage: "https://github.com/junegunn/fzf"
license: "MIT"

platforms:
  - os: windows
    arch: amd64
    archive: true
    url: "https://github.com/junegunn/fzf/releases/download/v0.46.1/fzf-0.46.1-windows_amd64.zip"

install:
  steps:
    - type: extract
      to: "{{ .TmpDir }}/fzf"
    - type: copy
      from: "{{ .TmpDir }}/fzf/fzf.exe"
      to: "{{ .BinDir }}/fzf.exe"
```

---

## License

This registry repository is licensed under MIT. Individual packages have their own licenses (see each manifest's `license` field).

---

## Related Projects

- [gbpm](https://github.com/Foggy-Forge/git-bash-package-manager) – The package manager CLI
- [scoop](https://scoop.sh/) – A Windows package manager (inspiration)
- [Homebrew](https://brew.sh/) – The Missing Package Manager for macOS