# fedora-packaging

A VSCode devcontainer setup for Fedora packaging.

# Example steps for updating the Haxe package

1. `kinit ${FAS_USERNAME}@FEDORAPROJECT.ORG`

2. `fedpkg clone haxe`

3. `cd haxe`

4. Edit `haxe.spec`
    - Update `Version:`
    - Reset `Release:` to `1%{?dist}`
    - Update the other build deps/rules if needed.
    - Add changelog in `%changelog`

5. `rpmdev-spectool -g haxe.spec`

6. `fedpkg new-sources haxe-*.tar.gz haxelib-*.tar.gz hx3compat-*.tar.gz`

7. `fedpkg mockbuild`

8. `fedpkg commit`

9. `fedpkg push`

10. `fedpkg build`
