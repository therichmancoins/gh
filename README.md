# gh
project_name: gh

release:
  prerelease: auto
  draft: true # we only publish after the Windows MSI gets uploaded
  name_template: "GitHub CLI {{.Version}}"

before:
  hooks:
    - go mod tidy
    - make manpages

builds:
  - <<: &build_defaults
      binary: bin/gh
      main: ./cmd/gh
      ldflags:
        - -s -w -X github.com/cli/cli/internal/build.Version={{.Version}} -X github.com/cli/cli/internal/build.Date={{time "2006-01-02"}}
        - -X main.updaterEnabled=cli/cli
    id: macos
    goos: [darwin]
    goarch: [amd64]

  - <<: *build_defaults
    id: linux
    goos: [linux]
    goarch: [386, arm, amd64, arm64]
    env:
      - CGO_ENABLED=0

  - <<: *build_defaults
    id: windows
    goos: [windows]
    goarch: [386, amd64]

archives:
  - id: nix
    builds: [macos, linux]
    <<: &archive_defaults
      name_template: "{{ .ProjectName }}_{{ .Version }}_{{ .Os }}_{{ .Arch }}{{ if .Arm }}v{{ .Arm }}{{ end }}"
    wrap_in_directory: true
    replacements:
      darwin: macOS
    format: tar.gz
    files:
      - LICENSE
      - ./share/man/man1/gh*.1
  - id: windows
    builds: [windows]
    <<: *archive_defaults
    wrap_in_directory: false
    format: zip
    files:
      - LICENSE

nfpms:
  - license: MIT
    maintainer: GitHub
    homepage: https://github.com/cli/cli
    bindir: /usr
    dependencies:
      - git
    description: GitHub’s official command line tool.
    formats:
      - deb
      - rpm
    files:
      "./share/man/man1/gh*.1": "/usr/share/man/man1"
