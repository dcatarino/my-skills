---
name: run-odoo-tests
description: This skill should be used when running Odoo unit tests in this codespace/dev environment — e.g. the user asks to run or verify tests for an Odoo module/addon, confirm a fix or PR passes, or set up a test database. Covers the odoo-bin command, the persistent test-DB workflow, targeting a specific test class or method, reading the pass/fail result, running long installs in the background, and fixing the "Address already in use" port conflict that happens when a dev server is already running.
version: 1.0.0
---

# Run Odoo unit tests

This environment runs Odoo from source. Tests run at module install/upgrade
time via `odoo-bin` with `--test-enable` and `--stop-after-init`.

## Environment facts

- Python:   `/home/odoo/.pyenv/shims/python3`
- odoo-bin: `/workspaces/odoo/odoo-bin`
- Config:   `.codespace-env/odoo.conf` (relative to the project root, e.g.
  `/workspaces/<project>`; or use the absolute path
  `/workspaces/<project>/.codespace-env/odoo.conf`). It holds `addons_path` and
  `db_user = odoo`.
- Postgres: local, `-h 127.0.0.1 -U odoo` (no password needed here).
- `MODULE` below = the addon you are testing (its technical name, e.g.
  `acetate_product`). Dependencies install automatically.

## Quick start (persistent reused DB — fastest iteration)

Pick a long-lived test DB name (e.g. `odoo-test-1`) and reuse it.

1. **Create + initialize the test DB once** (installs the module chain; odoo-bin
   creates the DB if it doesn't exist):

   ```bash
   /home/odoo/.pyenv/shims/python3 /workspaces/odoo/odoo-bin \
     -c .codespace-env/odoo.conf -d odoo-test-1 -i MODULE --stop-after-init
   ```

2. **Run the tests** (re-run this after each code change — no need to recreate
   the DB):

   ```bash
   /home/odoo/.pyenv/shims/python3 /workspaces/odoo/odoo-bin \
     -c .codespace-env/odoo.conf -d odoo-test-1 --test-enable -i MODULE \
     --stop-after-init --log-level=test
   ```

   `-i MODULE` re-runs the module's tests on every invocation even when the
   module is already installed (verified). `-u MODULE` is equivalent and also
   forces a code/schema reload — use it if a test isn't picking up a model/field
   change.

`--log-level=test` makes each test log a `Starting <Class>.<method> ...` line and
prints the final result summary.

## Read the result

The line to look for (success = `0 failed, 0 error(s)`):

```
... odoo.tests.result: 0 failed, 0 error(s) of 9 tests when loading database 'odoo-test-1'
```

When grepping a log, match both the summary and failure signatures so a crash
isn't mistaken for success:

```bash
grep -E "tests when loading|FAIL|ERROR:|AssertionError|Traceback" run.log | tail -40
```

## Run only specific tests

Use `--test-tags` to avoid running the whole module's suite:

```bash
# One test class
--test-tags /MODULE:TestClassName
# A single method
--test-tags /MODULE:TestClassName.test_05_something
```

(The leading `/` scopes the tag to this module.) Combine with the run command in
step 2.

## Long installs: run in the background

The first install (step 1) and full suites can take several minutes. Run them in
the background and watch the log rather than blocking:

```bash
... odoo-bin ... > /tmp/odoo_test.log 2>&1   # via Bash run_in_background
# then grep /tmp/odoo_test.log for "tests when loading" / "FAIL" / "Traceback"
```

## Gotcha: "Address already in use" (port conflict)

If a dev server is already running (check `ps aux | grep odoo-bin`), the test run
crashes at startup with:

```
OSError: [Errno 98] Address already in use
```

odoo-bin tries to bind the HTTP/gevent ports before loading modules, so **no
tests run**. Fix by giving the test run its own free ports (and disabling workers
/ cron for a clean, fast run):

```bash
--http-port=8970 --gevent-port=8971 --workers=0 --max-cron-threads=0
```

Add these flags to the run command. They don't affect test behavior — they just
avoid colliding with the running server.

## Test database management

odoo-bin creates the DB on `-i` if absent. To manage it manually:

```bash
createdb -h 127.0.0.1 -U odoo odoo-test-1     # fresh DB
dropdb   -h 127.0.0.1 -U odoo odoo-test-1     # reset / clean up
psql -h 127.0.0.1 -U odoo -lqt | cut -d'|' -f1   # list DBs
```

Recreate the DB from scratch only when you changed something install-time
(manifest deps, XML data, security) and want a clean state; for ordinary Python
test changes, just re-run step 2 against the existing DB.

## Notes on queue_job (OCA) in tests

Modules depending on `queue_job` enqueue jobs via `with_delay()`. Under tests
these are recorded as `queue.job` rows and not executed inline unless the test
sets `queue_job__no_delay=True` in context (or the env enables no-delay). To
assert a job was/wasn't enqueued, patch `with_delay` (e.g.
`unittest.mock.patch.object(type(record), "with_delay")`) and check the mock.
