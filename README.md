# com.visualstudio.code

> [!IMPORTANT]
> **This repository has moved and is no longer maintained.**
> Visual Studio Code is now built in the combined Flatpak repo:
> **[francoism90/flatpaks → src/com.visualstudio.code](https://github.com/francoism90/flatpaks/tree/main/src/com.visualstudio.code)**

## Switching to the new remote

Remove the old remote. `--force` uninstalls what came from it but keeps your data in `~/.var/app`:

```bash
flatpak remote-delete --user --force francoism90-vscode
```

Then add the new remote and reinstall:

```bash
flatpak remote-add --user --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
flatpak remote-add --user --if-not-exists flatpaks https://francoism90.github.io/flatpaks/index.flatpakrepo
flatpak install flatpaks com.visualstudio.code
```

Use the same scope you used originally. If you installed system-wide, drop `--user` and use `sudo`.

Issues and pull requests belong in [francoism90/flatpaks](https://github.com/francoism90/flatpaks).
