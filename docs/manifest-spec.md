# gbpm Manifest Specification

Version: 0.1.0 (Draft)

Manifests are YAML files that describe how to install a package using `gbpm`.

---

## Top-Level Fields

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
    checksum: "sha256:abc123..."  # optional

install:
  steps:
    - type: extract
      to: "{{ .TmpDir }}/fzf"
    - type: copy
      from: "{{ .TmpDir }}/fzf/fzf.exe"
      to: "{{ .BinDir }}/fzf.exe"
```

---

## Field Reference

### `name` (string, required)

The unique identifier for the package. Used when running `gbpm install <name>`.

**Rules:**
- Must be lowercase
- Alphanumeric characters and hyphens only
- Must match the directory name in the registry

**Examples:** `fzf`, `bat`, `ripgrep`, `my-tool`

---

### `version` (string, required)

The version of the package being installed.

**Recommendations:**
- Use semantic versioning (e.g., `1.2.3`)
- Match the upstream project's version
- Don't include `v` prefix

**Examples:** `0.46.1`, `1.0.0`, `2.3.4-beta`

---

### `description` (string, optional)

A short, human-readable description of the package.

**Example:** `"A general-purpose command-line fuzzy finder"`

---

### `homepage` (string, optional)

URL to the package's homepage or repository.

**Example:** `"https://github.com/junegunn/fzf"`

---

### `license` (string, optional)

The software license of the package.

**Examples:** `"MIT"`, `"Apache-2.0"`, `"BSD-3-Clause"`

---

## `platforms` (array, required)

A list of platform-specific download configurations. `gbpm` will select the first entry that matches the current OS and architecture.

### Platform Entry Fields

#### `os` (string, required)

Operating system identifier. For Git Bash on Windows, always use `windows`.

**Supported values:** `windows` (others like `linux`, `darwin` reserved for future use)

#### `arch` (string, required)

CPU architecture.

**Supported values:**
- `amd64` (64-bit Intel/AMD, most common)
- `386` (32-bit, rare)
- `arm64` (ARM 64-bit, future Windows on ARM)

#### `url` (string, required)

Direct download URL for the package asset.

**Requirements:**
- Must be HTTPS (preferred) or HTTP
- Should point to official release binaries
- Must be stable (no "latest" URLs that change content)

**Example:**
```yaml
url: "https://github.com/junegunn/fzf/releases/download/v0.46.1/fzf-0.46.1-windows_amd64.zip"
```

#### `archive` (boolean, optional, default: false)

Set to `true` if the downloaded file is an archive (`.zip`, `.tar.gz`, `.tar.bz2`, `.tar.xz`) that needs extraction.

Set to `false` (or omit) if the download is a standalone executable.

**Examples:**
```yaml
archive: true   # .zip file containing fzf.exe
archive: false  # direct .exe download
```

#### `checksum` (string, optional)

Checksum for verifying the downloaded file.

**Format:** `<algorithm>:<hexvalue>`

**Supported algorithms:** `sha256`, `sha512`, `md5`

**Example:**
```yaml
checksum: "sha256:e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855"
```

**Note:** Checksums are optional in version 0.1, but will be required in future versions for security.

---

## `install` (object, required)

Describes the installation steps.

### `install.steps` (array, required)

An ordered list of actions to perform during installation. Steps are executed sequentially.

---

## Installation Steps

### Step: `extract`

Extracts an archive to a specified directory.

**When to use:** When `archive: true` in the platform entry.

**Fields:**

- `type` (string, required): Must be `"extract"`
- `to` (string, required): Destination directory for extraction

**Supported archive formats:**
- `.zip` (primary format for Windows)
- `.tar.gz`, `.tgz`
- `.tar.bz2`, `.tbz2`
- `.tar.xz`, `.txz`

**Example:**
```yaml
- type: extract
  to: "{{ .TmpDir }}/fzf"
```

This extracts the downloaded archive into a temporary directory.

---

### Step: `copy`

Copies a file from one location to another.

**Fields:**

- `type` (string, required): Must be `"copy"`
- `from` (string, required): Source file path
- `to` (string, required): Destination file path

**Example:**
```yaml
- type: copy
  from: "{{ .TmpDir }}/fzf/fzf.exe"
  to: "{{ .BinDir }}/fzf.exe"
```

**Notes:**
- Parent directories are created automatically
- Overwrites existing files
- Preserves file permissions (on Unix-like systems)

---

## Template Variables

Steps can use Go template syntax to reference dynamic paths.

### Available Variables

| Variable | Description | Example Value |
|----------|-------------|---------------|
| `{{ .TmpDir }}` | Temporary directory for this installation | `/tmp/gbpm-install-12345` or `C:\Users\...\AppData\Local\Temp\gbpm-...` |
| `{{ .BinDir }}` | gbpm bin directory (install destination) | `C:\Users\Kris\.gbpm\bin` |
| `{{ .CacheDir }}` | gbpm cache directory | `C:\Users\Kris\.gbpm\cache` |
| `{{ .Home }}` | User's home directory | `C:\Users\Kris` |
| `{{ .GbpmHome }}` | gbpm home directory | `C:\Users\Kris\.gbpm` |

**Usage tips:**
- Always use `{{ .TmpDir }}` for intermediate extraction/processing
- Always use `{{ .BinDir }}` for final binary installation
- Use forward slashes (`/`) in paths; they work on Windows too

---

## Complete Examples

### Example 1: Simple Archive (fzf)

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

### Example 2: Direct Executable (hypothetical)

```yaml
name: hello
version: 1.0.0
description: "A simple hello world tool"
homepage: "https://example.com/hello"
license: "MIT"

platforms:
  - os: windows
    arch: amd64
    archive: false
    url: "https://example.com/releases/hello-1.0.0-windows-amd64.exe"

install:
  steps:
    - type: copy
      from: "{{ .TmpDir }}/hello-1.0.0-windows-amd64.exe"
      to: "{{ .BinDir }}/hello.exe"
```

**Note:** When `archive: false`, the downloaded file is placed directly in `{{ .TmpDir }}` with its original filename.

### Example 3: Multiple Binaries

```yaml
name: mytools
version: 2.1.0
description: "A collection of useful tools"
homepage: "https://example.com/mytools"
license: "Apache-2.0"

platforms:
  - os: windows
    arch: amd64
    archive: true
    url: "https://example.com/releases/mytools-2.1.0-windows-amd64.zip"

install:
  steps:
    - type: extract
      to: "{{ .TmpDir }}/mytools"
    - type: copy
      from: "{{ .TmpDir }}/mytools/tool1.exe"
      to: "{{ .BinDir }}/tool1.exe"
    - type: copy
      from: "{{ .TmpDir }}/mytools/tool2.exe"
      to: "{{ .BinDir }}/tool2.exe"
```

---

## Future Enhancements

The following features are planned for future versions:

### Additional Step Types

- `chmod`: Set file permissions (Unix-like systems)
- `rename`: Rename files
- `shell`: Execute shell commands (with restrictions)
- `symlink`: Create symbolic links
- `mkdir`: Create directories explicitly

### Additional Fields

- `dependencies`: List of packages this package depends on
- `conflicts`: List of packages that conflict with this one
- `post_install`: Commands to run after installation
- `pre_uninstall`: Commands to run before uninstall

### Validation

- Required checksums for security
- Schema validation on manifest submission
- Automated testing of manifests

---

## Validation Checklist

Before submitting a manifest:

- [ ] `name` matches directory name and is lowercase
- [ ] `version` is specified and matches upstream release
- [ ] `url` points to official release (not "latest")
- [ ] `archive` field correctly reflects file type
- [ ] All template variables use correct syntax
- [ ] Paths use forward slashes
- [ ] Tested with `gbpm install --file <manifest.yaml>`
- [ ] Binary runs successfully after installation
- [ ] (Optional) Checksum is included and verified

---

## Questions or Issues?

- Check the [main README](../README.md) for contribution guidelines
- Open an issue in the registry repository
- Refer to the [gbpm CLI documentation](https://github.com/Foggy-Forge/git-bash-package-manager)
