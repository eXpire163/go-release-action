# Go Release GitHub Action    

> Note: Original work at github.com/wangyoucao577/go-release-action, enable signify-openbsd by default

Automatically publish `Go` binaries to Github Release Assets through Github Action.    

## Features    

- support `signify-openbsd`

- check the original repo for more


## Usage

```yaml
# .github/workflows/release.yaml

on: 
  release:
    types: [created]

jobs:
  releases-matrix:
    name: Release Go Binary
    runs-on: ubuntu-latest
    strategy:
      matrix:
        # build and publish in parallel: linux/386, linux/amd64, windows/386, windows/amd64, darwin/amd64 
        goos: [linux, windows, darwin]
        goarch: ["386", amd64]
        exclude:  
          - goarch: "386"
            goos: darwin 
    steps:
    - uses: actions/checkout@v2
    - uses: chux0519/go-release-action@v2.1.0
      with:
        github_token: ${{ secrets.GITHUB_TOKEN }}
        # only needs to set following 2 secrets
        signify_sec_key: ${{ secrets.SIGNIFY_SEC_KEY }}
        signify_sec_key_pass: ${{ secrets.SIGNIFY_SEC_KEY_PASS }}
        md5sum: FALSE
        signify: TRUE
        goos: ${{ matrix.goos }}
        goarch: ${{ matrix.goarch }}
        extra_files: README.md
```

### Parameters

| Parameter | **Mandatory**/**Optional** | Description | 
| --------- | -------- | ----------- |
| github_token | **Mandatory** | Your `GITHUB_TOKEN` for uploading releases to Github asserts. |
| goos | **Mandatory** | `GOOS` is the running program's operating system target: one of `darwin`, `freebsd`, `linux`, and so on. |
| goarch | **Mandatory** | `GOARCH` is the running program's architecture target: one of `386`, `amd64`, `arm`, `s390x`, and so on. |
| goversion |  **Optional** | The `Go` compiler version. `1.16` by default, optional `1.13`, `1.14` or `1.15`. <br>It also takes download URL instead of version string if you'd like to use more specified version. But make sure your URL is `linux-amd64` package, better to find the URL from [Go - Downloads](https://golang.org/dl/).<br>E.g., `https://dl.google.com/go/go1.13.1.linux-amd64.tar.gz`. |
| project_path | **Optional** | Where to run `go build`. <br>Use `.` by default. |
| binary_name | **Optional** | Specify another binary name if do not want to use repository basename. <br>Use your repository's basename if not set. |
| pre_command | **Optional** | Extra command that will be executed before `go build`. You may want to use it to solve dependency if you're NOT using [Go Modules](https://github.com/golang/go/wiki/Modules). |
| build_command | **Optional** | The actual command to build binary, typically `go build`. You may want to use other command wrapper, e.g., [packr2](https://github.com/gobuffalo/packr/tree/master/v2), example `build_command: 'packr2 build'`. Remember to use `pre_command` to set up `packr2` command in this scenario.<br>It also supports the `make`(`Makefile`) building system, example `build_command: make`. In this case both `build_flags` and `ldflags` will be ignored since they should be written in your `Makefile` already. Also, please make sure the generated binary placed in the path where `make` runs, i.e., `project_path`. |
| executable_compression | **Optional** | Compression executable binary by some third-party tools. It takes compression command with optional args as input, e.g., `upx` or `upx -v`. <br>Only [upx](https://github.com/upx/upx) is supported at the moment.|
| build_flags | **Optional** | Additional arguments to pass the `go build` command. |
| ldflags | **Optional** | Values to provide to the `-ldflags` argument. |
| extra_files | **Optional** | Extra files that will be packaged into artifacts either. Multiple files separated by space. Note that extra folders can be allowed either since internal `cp -r` already in use. <br>E.g., `extra_files: LICENSE README.md` |
| md5sum | **Optional** | Publish `.md5` along with artifacts, `FALSE` by default. |
| sha256sum | **Optional** | Publish `.sha256` along with artifacts, `FALSE` by default. |
| signify | **Optional** | Publish `.sig` along with artifacts, `TRUE` by default. |
| signify_sec_key | **Optional** | Private key used by signify, pass this by secrets. |
| signify_sec_key_pass | **Optional** | Passphrase for private key protection |
| release_tag | **Optional** | Target release tag to publish your binaries to. It's dedicated to publish binaries on every `push` into one specified release page since there's no target in this case. DON'T set it if you trigger the action by `release: [created]` event as most people do.|
| overwrite | **Optional** | Overwrite asset if it's already exist. `FALSE` by default. |
| asset_name | **Optional** | Customize asset name if do not want to use the default format `${BINARY_NAME}-${RELEASE_TAG}-${GOOS}-${GOARCH}`. <br>Make sure set it correctly, especially for matrix usage that you have to append `-${{ matrix.goos }}-${{ matrix.goarch }}`. A valid example could be  `asset_name: binary-name-${{ matrix.goos }}-${{ matrix.goarch }}`. |
