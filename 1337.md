<div align="center">
  
# modification of `install.sh`  to comply with 1337 clusters

</div>

* **`case "${HOME:-}" ...` and `export HOME="$GOINFRE_DIR"**`: Temporarily tricks `install.sh` and its sub-installers (like `claude.ai/install.sh`) into treating `~/goinfre` as your home directory. Because `install.sh` re-downloads and restarts itself from GitHub (`dispatch_release`), the `case` check prevents it from accidentally setting `HOME` to `~/goinfre/goinfre` on restart.


* **`mkdir -p ...`**: Pre-creates the folders for binaries, caches, and temp files inside `~/goinfre` so the installer doesn't fail looking for missing directories.
* **`export TMPDIR="$GOINFRE_DIR/tmp"`**: Tells `mktemp` and `curl` to save temporary downloaded scripts in `~/goinfre/tmp` instead of `/tmp`.


* **`export UV_...`, `NPM_...`, and `XDG_...**`: Redirects all heavy Python (`uv`) environments, Node.js (`npm`) packages, and app data into `~/goinfre` instead of filling up your real `$HOME` quota.


* **`export PATH=...`**: Adds the `~/goinfre` binary folders to `PATH` so `install.sh` can find and verify the tools (`claude`, `opencode`, `fcc-server`) right after installing them.

```sh
#!/bin/sh
set -eu

# --- Redirect all installs, downloads, and caches to goinfre ---
case "${HOME:-}" in
    */goinfre) GOINFRE_DIR="$HOME" ;;
    *) GOINFRE_DIR="${HOME}/goinfre" ;;
esac

mkdir -p "$GOINFRE_DIR/tmp" \
         "$GOINFRE_DIR/.local/bin" \
         "$GOINFRE_DIR/.cache/uv" \
         "$GOINFRE_DIR/.cache/npm" \
         "$GOINFRE_DIR/.npm-global"

export HOME="$GOINFRE_DIR"
export TMPDIR="$GOINFRE_DIR/tmp"
export XDG_CACHE_HOME="$GOINFRE_DIR/.cache"
export XDG_DATA_HOME="$GOINFRE_DIR/.local/share"
export XDG_CONFIG_HOME="$GOINFRE_DIR/.config"
export UV_CACHE_DIR="$GOINFRE_DIR/.cache/uv"
export UV_TOOL_DIR="$GOINFRE_DIR/.local/share/uv/tools"
export UV_TOOL_BIN_DIR="$GOINFRE_DIR/.local/bin"
export UV_PYTHON_INSTALL_DIR="$GOINFRE_DIR/.local/share/uv/python"
export NPM_CONFIG_CACHE="$GOINFRE_DIR/.cache/npm"
export NPM_CONFIG_PREFIX="$GOINFRE_DIR/.npm-global"
export PATH="$GOINFRE_DIR/.local/bin:$GOINFRE_DIR/.npm-global/bin:$GOINFRE_DIR/.opencode/bin:$PATH"
# ---------------------------------------------------------------

```

### Then run the script normally:

```sh
sh install.sh

```

Because `sh install.sh` runs in a child process, changing `export HOME="$GOINFRE_DIR"` inside the script only affects the installer and its sub-installers (`claude.ai/install.sh`, `uv`, `npm`, `opencode`), leaving your current terminal's `$HOME` untouched.

---

### Expose the Installed Commands to Your Normal Shell

Because the installer placed all binaries and configs inside `~/goinfre` instead of `~`, add these lines to your real `/home/login/.zshrc` so your normal terminal can find `fcc-server`, `fcc-claude`, and `claude` and keep runtime caches in `goinfre`:

```sh
export PATH="$HOME/goinfre/.local/bin:$HOME/goinfre/.npm-global/bin:$HOME/goinfre/.opencode/bin:$PATH"
export TMPDIR="$HOME/goinfre/tmp"
export UV_CACHE_DIR="$HOME/goinfre/.cache/uv"
export UV_TOOL_DIR="$HOME/goinfre/.local/share/uv/tools"
export UV_TOOL_BIN_DIR="$HOME/goinfre/.local/bin"
export UV_PYTHON_INSTALL_DIR="$HOME/goinfre/.local/share/uv/python"
export CLAUDE_CONFIG_DIR="$HOME/goinfre/.claude"

```

Reload your shell configuration before running `fcc-server` or `fcc-claude`:

```sh
source ~/.zshrc

```
