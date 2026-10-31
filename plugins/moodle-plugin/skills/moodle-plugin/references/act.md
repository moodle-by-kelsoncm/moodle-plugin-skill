# Local validation with act

## Plugin CI
1. Copy `.github/workflows/ci.yml` to a scratch folder (do not change the repository copy) and adjust it:
   - **Matrix:** keep one combination: `- {php: '8.3', moodle-branch: 'MOODLE_503_STABLE', database: 'pgsql'}`.
   - **Dependencies:** in the *Initialise moodle-plugin-ci* step, add:
     ```bash
     sudo apt-get update -qq && sudo apt-get install -y -qq locales postgresql-client mariadb-client && sudo locale-gen en_AU.UTF-8 && mkdir -p $HOME/.nvm && printf '%s\n' 'export NVM_BIN=/opt/node22/bin' 'export PATH=/opt/node22/bin:$PATH' 'nvm() { :; }' > $HOME/.nvm/nvm.sh
     echo "NVM_DIR=$HOME/.nvm" >> $GITHUB_ENV
     ```
   - **Node:** a Docker volume `act-node22` with Node 22 (copied from the `node:22` image) replaces nvm. The act image ships Node 24, and Moodle requires 22.
     ```bash
     docker volume create act-node22
     docker run --rm -v act-node22:/opt/node22 node:22 cp -a /usr/local/. /opt/node22/
     ```
2. From the plugin folder:
   ```bash
   act workflow_dispatch -W <scratch>/ci.yml --pull=false \
     -P ubuntu-latest=catthehacker/ubuntu:act-latest \
     --container-options "-v act-node22:/opt/node22" > <scratch>/ci.log 2>&1
   ```
   Options and pitfalls:
   - `--pull=false` avoids Docker Hub authentication errors.
   - Git Bash on Windows: pass `-W "$(cygpath -w <scratch>/ci.yml)"`; `MSYS_NO_PATHCONV=1` breaks `-W`.
   - Run it in the background: it takes 5 to 6 minutes.
3. Log summary:
   ```bash
   grep -E "Success -|Failure -|Tests: " ci.log | sed 's/^.*pgsql\]   | //'
   ```
   To see the warnings of one step, use `sed -n '/Run Main Moodle Code Checker/,/Run Main Moodle PHPDoc/p'`.

## Shortcuts in the job container
After `act`, the job container (`act-CI-Moodle-MOODLE-503-STABLE--PHP-...`) stays up with Moodle and `node_modules` installed. Use it to iterate on JavaScript without reinstalling. The next `act` run recreates it.

```bash
C=$(docker ps --format '{{.Names}}' | grep '^act-CI-Moodle-MOODLE-503' | head -1)
P=~/<path>/<plugin>                         # plugin folder on the host
M=<plugin folder in the container>/moodle   # Linux/macOS: same as $P; Windows: /mnt/c/Users/<user>/...
docker exec $C rm -rf $M/public/mod/<x>/amd/src
docker cp $P/amd/src $C:$M/public/mod/<x>/amd/src
docker exec -w $M $C bash -lc 'export PATH=/opt/node22/bin:$PATH; npx eslint --fix public/mod/<x>/amd/src; npx eslint public/mod/<x>/amd/src --format unix'
docker exec -w $M $C bash -lc 'export PATH=/opt/node22/bin:$PATH; npx grunt amd --root=public/mod/<x>'
docker cp $C:$M/public/mod/<x>/amd/build $P/amd/build
```

Git Bash on Windows:
- `export MSYS_NO_PATHCONV=1` keeps container paths from being converted;
- if `docker cp` rejects the host path, use `"$(cygpath -w $P)"`.

For phpcs on changed files only, add a step to the scratch workflow: `cd moodle && $GITHUB_WORKSPACE/ci/vendor/bin/phpcs --standard=moodle <files>`.

## Converting legacy JavaScript to ES6
1. Wrap it in `export const init = (selector) => {...}` and read the data from `container.dataset.config`.
2. Run `eslint --fix`. It adds empty JSDoc blocks.
3. Replace `var` with `let` and run eslint again: `no-undef` points to uses outside the block.
4. Give types and descriptions to every `@param`.
5. Split functions with complexity above 20.
6. Remove `console`, `confirm` and globals (`M.cfg` becomes `core/config`; `bootstrap` becomes jQuery).
7. Check the indentation after `--fix`.
8. For large edits, write a script file and run it; inline Python in a shell heredoc can turn `"\\b"` and `"\\r"` into control characters, and `sed` replacements turn `\f` (as in `\form\`) into a form feed. Check namespaces after scripted edits. Some editing tools also turn `\uXXXX` written in a file into the character itself: build it with `chr(92)` in PHP tests and check JSON escapes in the documentation.

## Documentation
```bash
act workflow_dispatch -W .github/workflows/docs.yml -j build-docs --pull=false -P ubuntu-latest=catthehacker/ubuntu:act-latest
```
`sphinx-build -W` fails on any warning (a page outside the toctree, a missing image).

## Throwaway Moodle for browser tests
1. Write a `docker-compose.yml` in a scratch folder:
   - Postgres and a Moodle 5.3 image, on its own port (for example 8091).
   - Mount only the plugin, at `/var/www/html/public/mod/<x>`: Moodle 5.1+ serves from `public/`, and old compose files that mount at `/var/www/html/mod/...` do not load the plugin.
2. Install the site (or let the image entrypoint do it) and wait for `install_database.php` to finish (`pgrep -f install_database.php` inside the container; on the host, a wait loop with `pgrep -f "<text>"` matches its own command line).
   - Remounting a plugin recreates the container: run `admin/cli/upgrade.php` and `admin/cli/purge_caches.php` afterwards. A plugin without `amd/build` gives a RequireJS 404 for its module; build it in the job container first.
3. Create test data with a CLI PHP script (`testing_util::get_data_generator()`): course, activities, plugin instance, files and restrictions. Copy it with `docker cp` and run it as `www-data`.
4. Test in a browser.
   - Log in with the throwaway site's test account.
   - Synthetic mouse events on canvases need pauses between them, because modals and services are asynchronous.
   - Check the result in the database (`psql` in the Postgres container) and that the console has no errors.
5. Take documentation screenshots there. For the student view, use "Log in as": `/course/loginas.php?id=<course>&user=<id>&sesskey=...`.
