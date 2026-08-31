# python-gate

**This is a GitHub template.** Use it as *Use this template*, or:

```bash
nix flake new -t github:Hedronite/python-gate ./my-app
```

Nix is the gate. uv owns the expression. `nix run` is the product.

`mkVirtualEnv` / `withPackages` builds a world; `mkApplication` / `wrapProgram` locks the door; nix-ld is the FHS lie (unsafe, Level 2).

A Python shop that is not on NixOS still `uv sync`s from `uv.lock`. A Nix user or CI `nix develop`s and `nix run .#`s the same closure. Two proofs: `uv.lock` (packages) and `flake.lock` (builders). **A lockfile is not a closure.**

This repository is the working example *and* the cloneable template. v1 is Level 3: no CUDA, no PyO3, no native extensions.

## The ladder

| Level | What you have | What you do not have |
| --- | --- | --- |
| 0 | Host Python, `pip install` | A lock, a world, a product |
| 1 | `uv.lock` / `poetry.lock` | A closure. Builders are the host |
| 2 | nix-ld / steam-run / `LD_LIBRARY_PATH` | Bit-identity. This is the FHS lie |
| **3** | **uv2nix `mkVirtualEnv` + `mkApplication`. This template** | CUDA, PyO3, native extensions |
| 4 | Native/FFI bits patched into the store | (out of scope for v1) |

Level 2 can run a wheel. It is not a closed world. Do not claim it is bit-identical.

## Init

```bash
# GitHub template, or:
nix flake new -t github:Hedronite/python-gate ./my-app
cd ./my-app

# Rename the project: pyproject.toml [project] name, [project.scripts],
# src/<module>/, and `projectName` in flake.nix. Then:
uv lock
```

Without Nix (inner loop):

```bash
uv sync
uv run python-gate
```

With Nix (same world, locked door):

```bash
nix develop          # store venv + peace treaty
nix run .#           # the product; network-free
```

## Add a dependency

uv writes the expression. Nix consumes it. Do not hand-translate `uv.lock` into `python3Packages`.

```bash
uv add httpx         # or any pure-Python wheel uv2nix can eat
git add uv.lock pyproject.toml
nix run .#           # rebuilds the world from the new lock
git add flake.lock   # only if flake inputs moved
git commit
```

Commit **both** lockfiles. `uv.lock` pins packages. `flake.lock` pins builders. Either one alone is a story, not a closure.

## Peace treaty (dev shell)

The Nix shell and uv share one interpreter. They do not share authority over the environment.

| Variable | Value | Why |
| --- | --- | --- |
| `UV_NO_SYNC` | `1` | uv must not provision a second venv |
| `UV_PYTHON` | the Nix interpreter | one Python set |
| `UV_PYTHON_DOWNLOADS` | `never` | Nix owns the interpreter |
| `PYTHONPATH` | unset in `shellHook` | kill nixpkgs `PYTHONPATH` leakage |

On NixOS, if you must run an **unpatched** FHS binary, set `NIX_LD` and `NIX_LD_LIBRARY_PATH` (nix-ld). **Never set `LD_LIBRARY_PATH` as the default.** That is a Level 2 leak into a Level 3 world.

## Must not

- Keep a second venv (`.venv`) beside the store venv. `uv run` inside `nix develop` is how you grow a second world.
- Use poetry2nix. It is unmaintained. This template is uv + uv2nix.
- Claim Level 2 (nix-ld / `LD_LIBRARY_PATH`) is bit-identical.
- Edit `uv.lock` or `flake.lock` by hand.
- Hand-translate `uv.lock` into `python3Packages`.
- Set `LD_LIBRARY_PATH` as the default on NixOS.

## Agent grading

A change is done when all of these pass:

1. **Flake eval.** `nix flake show` lists `packages.<system>.default`, `devShells.<system>.default`, and `templates.default`.
2. **Import inside the declared env.** `nix develop --command python -c "import python_gate, rich"` succeeds.
3. **`uv sync --locked`.** The non-Nix inner loop still reproduces from `uv.lock`.
4. **Same closure.** `nix run .#` and `nix develop` use the same Python set (same interpreter, same uv2nix overlay). The product is the wrapped application, not a hope.

CI on `ubuntu-latest` runs `nix build` / `nix run` (Determinate Nix) and `uv sync --locked` (setup-uv). Green means both proofs held, not that one was skipped.

## Example app

`src/python_gate` is a console script plus `rich`. `nix run .#` prints versions and `square(12)` — a pure function, no HTTP. That is the point: the closure is enough.

## Further reading

Short doctrine: [`docs/theses.md`](docs/theses.md).

Upstream builders (2026 map — do not use last year's):

- [uv](https://docs.astral.sh/uv/)
- [uv2nix](https://pyproject-nix.github.io/uv2nix/)
- [pyproject.nix](https://pyproject-nix.github.io/pyproject.nix/)
- [pyproject-build-systems](https://github.com/pyproject-nix/build-system-pkgs)

## License

MIT. Copyright Hedronite 2026.
