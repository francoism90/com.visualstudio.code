# Visual Studio Code Flatpak<!-- omit in toc -->

🚨 Warning: This is an unofficial Flatpak build of Visual Studio Code, generated from the official Microsoft-built [.deb packages](https://code.visualstudio.com/download). Use it at your own risk, it is recommended to build it yourself.

## Table of Contents<!-- omit in toc -->

- [Quick Start](#quick-start)
  - [Build](#build)
    - [Nested-sandbox install error](#nested-sandbox-install-error)
  - [Signing](#signing)
- [Usage](#usage)
  - [Execute commands in the host system](#execute-commands-in-the-host-system)
  - [Use host shell in the integrated terminal](#use-host-shell-in-the-integrated-terminal)
  - [Support for language extension](#support-for-language-extension)
- [Support](#support)

## Quick Start

Add the remote repository:

```bash
flatpak remote-add --user --if-not-exists francoism90-vscode https://francoism90.github.io/com.visualstudio.code/index.flatpakrepo
```

Update the repository:

```bash
flatpak update
```

Install the app:

```bash
flatpak install francoism90-vscode com.visualstudio.code
```

> Note: the app will automatically update when you run `flatpak update`.

```bash
flatpak run com.visualstudio.code
```

### Build

It is possible to build the app yourself instead of using the prebuilt, signed
repo above.

```bash
git clone https://github.com/francoism90/com.visualstudio.code.git
cd com.visualstudio.code
./build.sh
flatpak run com.visualstudio.code
```

`build.sh` adds the Flathub remote (user), builds to a local repo, then
installs from that repo with the host's own `flatpak` binary (see
[Nested-sandbox install error](#nested-sandbox-install-error) below for why
it's split into two steps).

To build manually:

```bash
flatpak run org.flatpak.Builder --user --install-deps-from=flathub --force-clean --repo=repo \
  build-dir com.visualstudio.code.yaml
flatpak --user remote-add --if-not-exists com.visualstudio.code-local ./repo --no-gpg-verify
flatpak --user install --noninteractive com.visualstudio.code-local com.visualstudio.code
```

#### Nested-sandbox install error

If `flatpak-builder` on your `$PATH` is itself a Flatpak (`org.flatpak.Builder`,
e.g. on immutable/hardened distros without a native package), running it with
`--install` directly can fail with:

```text
bwrap: No permissions to create a new namespace, likely because the kernel
does not allow non-privileged user namespaces.
Error: Failed to install com.visualstudio.code: ...
```

That's `org.flatpak.Builder`'s own sandbox trying to nest another `bwrap`
sandbox for `flatpak install`, which some kernels/hardening policies block
regardless of user-namespace permissions otherwise being fine. Building to a
local repo and installing with the _host's_ `flatpak` binary — as `build.sh`
and the manual steps above do — sidesteps it, since that install then only
needs one level of sandboxing, not two.

### Signing

The signed repo published to GitHub Pages by `.github/workflows/flatter.yml`
needs a GPG key in the `GPG_PRIVATE_KEY` (and optionally `GPG_PASSPHRASE`)
repo secrets. Generate one with:

```bash
bin/create-keys "Your Name" "you@example.com"
```

This prints the values to add as repo secrets. Delete `private.key` and the
`flatter-keyring/` directory afterwards — never commit them.

## Usage

Most functionality works out of the box, though please note that flatpak runs in an isolated environment and some work is necessary to enable those features.

### Execute commands in the host system

To execute commands on the host system, run inside the sandbox:

`flatpak-spawn --host <COMMAND>`

or

`host-spawn <COMMAND>`

- Most users seem to report a better experience with `host-spawn`

### Use host shell in the integrated terminal

Another option to execute commands is to use your host shell in the integrated terminal instead of the sandbox one.

For that go to `File -> Preferences -> Settings` and find `Features > Terminal > Integrated > Profiles`, then click on `Edit in settings.json` (The important thing here is to open settings.json)

And make sure that you have the following lines there:

**Using `flatpak-spawn`:**

```json
{
  "terminal.integrated.defaultProfile.linux": "bash",
  "terminal.integrated.profiles.linux": {
    "bash": {
      "path": "/usr/bin/flatpak-spawn",
      "args": ["--host", "--env=TERM=xterm-256color", "bash"],
      "icon": "terminal-bash",
      "overrideName": true
    }
  }
}
```

**Using `host-spawn`:**

```json
{
  "terminal.integrated.defaultProfile.linux": "bash",
  "terminal.integrated.profiles.linux": {
    "bash": {
      "path": "/app/bin/host-spawn",
      "args": ["bash"],
      "icon": "terminal-bash",
      "overrideName": true
    }
  }
}
```

- You can change **bash** to any terminal you are using: zsh, fish, sh.
- `overrideName` allows for the 'name' (or whatever you set it to) of the shell you're using to appear (e.g. normally zsh, fish, sh).

### Support for language extension

Some Visual Studio extensions depend on packages that might exist on your host, but they are not accessible through Flatpak. Like support for programming languages: gcc, python, etc.

**See available SDK:**

```bash
flatpak run --command=sh com.visualstudio.code
ls /usr/bin # shared runtime
ls /app/bin # bundled with this flatpak
```

**Getting support for additional languages, you have to install SDK extensions, e.g.:**

```bash
flatpak install flathub org.freedesktop.Sdk.Extension.dotnet
flatpak install flathub org.freedesktop.Sdk.Extension.golang
FLATPAK_ENABLE_SDK_EXT=dotnet,golang flatpak run com.visualstudio.code
```

**Container support (Podman):**

To use Podman as a container runtime inside the sandbox (e.g. for Dev Containers), install the [`org.freedesktop.Sdk.Extension.podman`](https://github.com/francoism90/org.freedesktop.Sdk.Extension.podman) SDK extension from its own repo (not on Flathub — see that repo for why) and enable it the same way:

```bash
flatpak remote-add --if-not-exists francoism90-podman https://francoism90.github.io/org.freedesktop.Sdk.Extension.podman/index.flatpakrepo
flatpak install francoism90-podman org.freedesktop.Sdk.Extension.podman
FLATPAK_ENABLE_SDK_EXT=podman flatpak run com.visualstudio.code
```

**Finding other SDK:**

`flatpak search <TEXT>`
