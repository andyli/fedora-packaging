# fedora-packaging

A VSCode devcontainer setup for Fedora packaging.

# WSL config

Remember to run `ssh-add` in WSL such that ssh agent will work in devcontainer.

# Example steps for updating the Haxe package

1. `kinit ${FAS_USERNAME}@FEDORAPROJECT.ORG`

2. `cd projects`

3. `fedpkg clone haxe`

4. `cd haxe`

5. Edit `haxe.spec`
    - Update `Version:`
    - Reset `Release:` to `1%{?dist}`
    - Update the other build deps/rules if needed.
    - Add changelog in `%changelog`

6. `rpmdev-spectool -g haxe.spec`

7. `fedpkg new-sources haxe-*.tar.gz haxelib-*.tar.gz hx3compat-*.tar.gz`

8. `fedpkg mockbuild`

9. `fedpkg commit`

10. `fedpkg push`

11. `fedpkg build`
