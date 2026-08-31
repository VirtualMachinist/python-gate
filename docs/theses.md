# Twelve theses

1. **The Python environment is part of the program.** An interpreter, a set of wheels, and a script are one artifact. Shipping the script alone is a wish.

2. **A lockfile is not a closure.** `uv.lock` pins packages. `flake.lock` pins builders. The closure is the store paths those two proofs evaluate to.

3. **uv owns the expression.** Add, remove, and pin through `pyproject.toml` / `uv.lock`. Do not hand-translate the lock into `python3Packages`. Do not edit lockfiles by hand.

4. **Nix is the gate.** Nothing enters the world except through evaluation. If it does not evaluate, it is not a dependency.

5. **`mkVirtualEnv` / `withPackages` builds a world.** Individual packages are not a product. The aggregated environment is.

6. **`mkApplication` / `wrapProgram` locks the door.** The virtualenv is an implementation detail. The product is a wrapped binary.

7. **Wrapping is FFI.** The ABI of the program is the store path, not `LD_LIBRARY_PATH`, not the host `libstdc++`, not a hope that the wheel matches the machine.

8. **Eval is the critic.** Tests that run against a second venv are reviewing a different play. `nix flake show`, import inside the declared env, `nix run`.

9. **The product is `nix run`.** A green `uv sync` is a proof of packages. The product is the closed world a stranger can run without becoming you.

10. **nix-ld is the FHS lie.** It is Level 2: useful for unpatched binaries, unsafe as a default, and never bit-identical. On NixOS use `NIX_LD` / `NIX_LD_LIBRARY_PATH`. Never set `LD_LIBRARY_PATH` as the default.

11. **One Python set.** `devShells.default` and `packages.default` share the same interpreter and the same uv2nix overlay. Editables on the shell are a lens, not a second world. The peace treaty (`UV_NO_SYNC`, `UV_PYTHON`, `UV_PYTHON_DOWNLOADS=never`, unset `PYTHONPATH`) is how they stay one.

12. **This template is Level 3.** No CUDA, no PyO3, no native extension in v1. Level 4 is a different door. Do not pretend Level 2 is this one.
