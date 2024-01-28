# Initializ Development Environment (IDE)

## Getting started

### Docker

- Start the server:
```bash
docker run -it --init -p 3000:3000 -v "$(pwd):/home/workspace:cached" initializ/ide
```
- Visit the URL printed in your terminal.


_Note_: Feel free to use the `nightly` tag to test the latest version, i.e. `initializ/ide:nightly`.

#### Custom Environment
- If you want to add dependencies to this Docker image, here is a template to help:
	```Dockerfile

	FROM initializ/ide:latest

	USER root # to get permissions to install packages and such
	RUN # the installation process for software needed
	USER ide # to restore permissions for the web interface

	```
- For additional possibilities, please consult the `Dockerfile` for IDE at https://github.com/initializ/ide-releases/

#### Pre-installing VSCode extensions

You can pre-install vscode extensions in such a way:

```dockerfile
FROM initializ/ide:latest

ENV IDE_ROOT="/home/.ide"
ENV IDE="${IDE_ROOT}/bin/ide"

SHELL ["/bin/bash", "-c"]
RUN \
    # Direct download links to external .vsix not available on https://open-vsx.org/
    # The two links here are just used as example, they are actually available on https://open-vsx.org/
    urls=(\
        https://github.com/rust-lang/rust-analyzer/releases/download/2022-12-26/rust-analyzer-linux-x64.vsix \
        https://github.com/VSCodeVim/Vim/releases/download/v1.24.3/vim-1.24.3.vsix \
    )\
    # Create a tmp dir for downloading
    && tdir=/tmp/exts && mkdir -p "${tdir}" && cd "${tdir}" \
    # Download via wget from $urls array.
    && wget "${urls[@]}" && \
    # List the extensions in this array
    exts=(\
        # From https://open-vsx.org/ registry directly
        10nates.ollama-autocoder \
        # From filesystem, .vsix that we downloaded (using bash wildcard '*')
        "${tdir}"/* \
    )\
    # Install the $exts
    && for ext in "${exts[@]}"; do ${IDE} --install-extension "${ext}"; done
```

### Linux

- [Download the latest release](https://github.com/initializ/ide/releases/latest)
- Untar and run the server
	```bash
	tar -xzf ide-v${IDE_VERSION}.tar.gz
	cd ide-v${IDE_VERSION}
	./bin/ide # you can add arguments here, use --help to list all of the possible options
	```

  From the possible entrypoint arguments, the most notable ones are
	- `--port` - the port number to start the server on, this is 3000 by default
	- `--without-connection-token` - used by default in the docker image
	- `--connection-token` & `--connection-token-file` for securing access to the IDE, you can read more about it in [Securing access to your IDE](#securing-access-to-your-ide).
	-  `--host` - determines the host the server is listening on. It defaults to `localhost`, so for accessing remotely it's a good idea to add `--host 0.0.0.0` to your launch arguments.

- Visit the URL printed in your terminal.

_Note_: You can use [pre-releases](https://github.com/initializ/ide/releases) to test nightly changes.

### Securing access to your IDE

Since IDE v1.64, you can access the Web UI without authentication (anyone can access the IDE using just the hostname and port), if you need some kind of basic authentication then you can start the server with `--connection-token YOUR_TOKEN`, the provided `YOUR_TOKEN` will be used and the authenticated URL will be displayed in your terminal once you start the server. You can also create a plaintext file with the desired token as its contents and provide it to the server with `--connection-token-file YOUR_SECRET_TOKEN_FILE`.

If you want to use a connection token and are working with IDE via [the Docker image](https://hub.docker.com/r/initializ/ide), you will have to edit the `ENTRYPOINT` in [the Dockerfile](https://github.com/initializ/ide-releases/blob/main/Dockerfile) or modify it with the [`entrypoint` option](https://docs.docker.com/compose/compose-file/compose-file-v3/#entrypoint) when working with `docker-compose`.

### Deployment guides

Please refer to [Guides](https://github.com/initializ/ide/tree/docs/guides) to learn how to deploy IDE to your cloud provider of choice.

## The scope of this project

This project only adds minimal bits required to run VS Code in a server scenario. We have no intention of changing VS Code in any way or to add additional features to VS Code itself. Please report feature requests, bug fixes, etc. in the upstream repository.

> **For any feature requests, bug reports, or contributions that are not specific to running VS Code in a server context, please go to [Visual Studio Code - Open Source "OSS"](https://github.com/microsoft/vscode)**

## Documentation

All documentation is available in [the `docs` branch](https://github.com/initializ/ide/tree/docs) of this project.

## Supporters

The project is supported by companies such as [initializ](https://initializ.ai/)

## Contributing

Thanks for your interest in contributing to the project 🙏.

To learn about the code structure and other topics related to contributing, please refer to the [development docs](https://github.com/initializ/ide/blob/docs/development.md).

## Bundled Extensions

VS Code includes a set of built-in extensions located in the [extensions](extensions) folder, including grammars and snippets for many languages. Extensions that provide rich language support (code completion, Go to Definition) for a language have the suffix `language-features`. For example, the `json` extension provides coloring for `JSON` and the `json-language-features` extension provides rich language support for `JSON`.

## Development Container

This repository includes a Visual Studio Code Dev Containers / GitHub Codespaces development container.

- For [Dev Containers](https://aka.ms/vscode-remote/download/containers), use the **Dev Containers: Clone Repository in Container Volume...** command which creates a Docker volume for better disk I/O on macOS and Windows.
     - If you already have VS Code and Docker installed, you can also click [here](https://vscode.dev/redirect?url=vscode://ms-vscode-remote.remote-containers/cloneInVolume?url=https://github.com/microsoft/vscode) to get started. This will cause VS Code to automatically install the Dev Containers extension if needed, clone the source code into a container volume, and spin up a dev container for use.
- For Codespaces, install the [GitHub Codespaces](https://marketplace.visualstudio.com/items?itemName=GitHub.codespaces) extension in VS Code, and use the **Codespaces: Create New Codespace** command.

Docker / the Codespace should have at least **4 Cores and 6 GB of RAM (8 GB recommended)** to run full build. See the [development container README](.devcontainer/README.md) for more information.

## Legal
This project is not affiliated with Microsoft Corporation.
