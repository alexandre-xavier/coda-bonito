# coda-bonito

Java and Groovy code style rules for [Claude Code](https://claude.com/claude-code), packaged as the `coda` plugin with one skill, `coda-bonito`.

The skill covers parameter ordering, naming, blank lines, primitives vs wrappers, `var`, dead-code cleanup, safe logging, constants, lambdas, and Spock/JUnit test conventions.

## Install

In Claude Code:

```
/plugin marketplace add alexandre-xavier/coda-bonito
/plugin install coda@coda-bonito
```

## Use

- **Automatic:** Claude loads the skill on its own when you write, refactor, or review Java or Groovy code.
- **Manual:** type `/coda:coda-bonito`.

> **Renamed in 0.3.0:** the skill `bonito` is now `coda-bonito`, so the manual command changed from `/coda:bonito` to `/coda:coda-bonito`. There is no alias. After updating, run `/reload-plugins` or start a new session, and update any prompts or docs that reference `/coda:bonito`.

## Update

```
/plugin marketplace update coda-bonito
/plugin update coda@coda-bonito
```

## Repository layout

```
.claude-plugin/
  marketplace.json   # marketplace "coda-bonito", lists the plugin
  plugin.json        # plugin "coda", version
skills/
  coda-bonito/SKILL.md  # the rules
CHANGELOG.md
```

## Contributing / versioning

- `develop` is where work happens. `main` holds released versions. Installs track `main`.
- Commits follow [Conventional Commits](https://www.conventionalcommits.org/) (`feat:`, `fix:`, `docs:`).
- Versions follow [SemVer](https://semver.org/). On each release, bump the version in `skills/coda-bonito/SKILL.md`, `.claude-plugin/plugin.json`, and `.claude-plugin/marketplace.json`. Then add a `CHANGELOG.md` entry and tag `vX.Y.Z`.
