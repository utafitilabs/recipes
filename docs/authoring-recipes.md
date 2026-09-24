# Add / update a recipe

Edit the files under `<vendor>/<package>/<version>/`, push to `main`; the
Action regenerates `flex/main`. Recipe contents must stay client-agnostic
(synthetic example domains only) — same rule as the bundles themselves.

A recipe carries what a host would otherwise have to write by hand: the bundle
registration (`bundles`), the module's `config/packages/<alias>.yaml` and
`config/routes/<alias>.yaml` (shipped under the recipe's `config/` and copied by
`copy-from-recipe`), and any `.env` block the module documents (`env`; keys
named `#1`, `#2`, … become comment lines, in order).

### What decides those three keys

`manifest.json` is JSON and carries no comments, so the citations live here.
The three configurators a uhifadhi recipe uses are documented in the upstream
[`symfony/recipes` README](https://github.com/symfony/recipes/blob/main/README.rst):

- **`bundles`** — "Enables one or more bundles in the Symfony application by
  appending them to the `bundles.php` file. Its value is an associative array
  where the key is the bundle class name and the value is an array of
  environments where it must be enabled."
- **`copy-from-recipe`** — "It's identical to `copy-from-package` but contents
  are copied from the recipe itself instead of from the Composer package
  contents."
- **`env`** — "Adds the given list of environment variables to the `.env` file
  stored in the root of the Symfony project."

The **comment keys are not in that README**; Flex's own source is the warrant,
and it is exact about the shape — a `#` followed by digits, nothing else:

```php
if ('#' === $key[0] && is_numeric(substr($key, 1))) {
    if ('' === $value) { $data .= "#\n"; } else { $data .= '# '.$value."\n"; }
    continue;
}
```

— `vendor/symfony/flex/src/Configurator/EnvConfigurator.php`. So `"#1"`, `"#2"`
work and `"#"` or `"#note"` do not: an unnumbered key falls through and is
written as a variable assignment.

## Recipes the skeleton itself installs

`uhifadhi/seam-module` and `uhifadhi/shell-module` are shipped by the skeleton
([`uhifadhi/uhifadhi`](https://github.com/utafitilabs/uhifadhi)), so their
recipes have a second half: the skeleton's `symfony.lock` records the recipe
version, its hash and the files it tracks, and that ledger is what tells a fresh
`create-project` there is nothing to update. Change one of those two recipes —
editing bytes in place or adding a version — and re-sync the ledger in the
skeleton:

```bash
composer recipes:update uhifadhi/seam-module
```

Skip it and every new installation is told "update available" on first boot,
with nothing to apply, because the stored hash no longer matches the recipe.
A file hand-committed into the skeleton without that command is the same bug.
