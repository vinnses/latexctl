# latexctl

`latexctl` is a small operational toolkit for LaTeX projects that use TeX Live.
It helps a project bootstrap `tex/`, detect source dependencies, synchronize
missing TeX packages with `tlmgr`, classify build failures, retry recoverable
builds, and package source files for export.

This project was extracted from the tooling built during the creation of
`template-docker-latex`. It was developed with Codex through a long interactive
process: small scripts became working conventions, rough ideas were tested
against real LaTeX failures, and repeated fixes were condensed into the commands
kept here. The goal is not to present a generic one-shot "AI-generated script
bundle", but to preserve the practical tooling that emerged from using,
breaking, revising, and refining a LaTeX Docker workflow over time.

## Status

Early extraction. The current implementation is intentionally close to the
working project-local version it came from. The public interface may still be
refined as `latexctl` is integrated back into `template-docker-latex`,
`vibeserver`, and thesis-style projects.

## Commands

```bash
bin/latexctl help
bin/latexctl bootstrap
bin/latexctl sync
bin/latexctl build [tex/main.tex] [latexmk args...]
bin/latexctl classify-error --log tex/main.log --output-dir .latex-errors
bin/latexctl report-build --status success --log tex/main.log --output build-report.md
bin/latexctl ctan-mirror
bin/latexctl ziptex
```

## Project Model

By default, `latexctl` operates on the current project directory. A project is
expected to keep LaTeX sources under:

```text
tex/
```

You can target another project explicitly:

```bash
LATEXCTL_PROJECT_ROOT=/path/to/project /path/to/latexctl/bin/latexctl sync
```

The tool writes project-local state such as:

```text
.used_packages
.used_tools
.latex-errors/
build-report.md
```

## Dependency Sync

`latexctl sync` scans `.tex`, `.cls`, and `.sty` files under `tex/`, detects
`\documentclass`, `\usepackage`, and `\RequirePackage`, and installs missing
packages through TeX Live.

It prefers user-mode installs for relocatable TeX packages:

```bash
tlmgr --usermode install <package>
```

For non-relocatable tools listed in `extra-tools.txt`, it uses:

```bash
sudo tlmgr install <tool>
```

`latexctl` should not silently erase or reinstall a TeX Live tree. Runtime
bootstrap of TeX Live itself belongs to the surrounding container or host setup.

## Missing File Recovery

When a build fails because TeX cannot find a file such as `logreq.sty`,
`latexctl build` can classify the failure, resolve the owning TeX Live package,
install it, and retry the build in a bounded way.

Local project files are treated differently from TeX environment files. A missing
`diagram.png` or `chapters/intro.tex` is classified as a user/project problem,
not as something `tlmgr` should fix.

## CTAN Mirror Selection

`bin/latexctl ctan-mirror` resolves the mirror in this order:

1. `CTAN_MIRROR`
2. the first non-comment line in `ctan-mirrors.txt`
3. `https://mirror.ctan.org/systems/texlive/tlnet`

## Tests

Run the shell tests from the repository root:

```bash
tests/test_latexctl.sh
tests/test_sync_tlmgr.sh
```

The tests use mocked `tlmgr`, `kpsewhich`, `sudo`, and `latexmk` binaries where
needed. They should not install TeX packages on the host.

## License

MIT.
