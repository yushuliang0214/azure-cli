# Azure CLI — PR Review Manual

A reviewer's manual for `Azure/azure-cli` PRs, synthesized from the repo `doc/` tree.
**Part 1** is a fast checklist; each group links to the matching detail in **Part 2**.

- Guidelines apply to **both command modules and extensions**. Items marked **(\*)** are command-module-only.
- Local validation: `azdev style <module>`, `azdev linter <module>`, `azdev test <module>`.

---

# Part 1 — Review Checklist

### [0. Review workflow & local validation](#0-review-workflow-and-local-validation)
- [ ] CI is green (style/pylint/pep8, linter, tests).
- [ ] Changes validated with `azdev style|linter|test <module>`.
- [ ] Code supports Python 3.10–3.14; no Python-version-specific breakage.

### [1. PR hygiene](#1-pr-hygiene)
- [ ] Title starts with `[Component]` (customer-facing → `HISTORY.rst`) or `{Component}` (not customer-facing).
- [ ] `Core` component targets `azure-cli-core/HISTORY.rst`; otherwise `azure-cli/HISTORY.rst`.
- [ ] Breaking change → `BREAKING CHANGE:` as 2nd title part; hotfix → `Hotfix`; issue fix → `Fix #<n>`.
- [ ] Verb is present-tense base form, capitalized: Add / Change / Deprecate / Remove / Fix.
- [ ] Multiple/override notes placed under `History Notes` in the description.
- [ ] Hotfix PR branches off `release`; merge-back to `dev` is a **merge commit, not squash**.

### [2. Command naming & behavior](#2-command-naming-and-behavior)
- [ ] Follows `[noun] [noun] [verb]`; every command has a verb.
- [ ] Multi-word subgroups hyphenated; no needless hyphenated names that a subgroup would fix.
- [ ] No empty/single-command subgroups adding tree depth (unless siblings are coming).
- [ ] Child collections use `CREATE`/`DELETE` **or** `ADD`/`REMOVE`.
- [ ] Standard verbs behave correctly: `create` idempotent+returns resource; `update` uses `generic_update_command`; `show` uses `show_command`/`custom_show_command` (404→exit 3); `delete` returns nothing; `list` is one command with args; `wait` present if any `--no-wait`.
- [ ] (\*) No confusing single-word verbs (`get`≈show, `new`≈create).

### [3. Argument conventions](#3-argument-conventions)
- [ ] No units in arg names (units in help); `-id` ⇒ GUID; ARM-ID args use "name or ID" overload.
- [ ] Canonical names used (`resource_group_name`, not `rg`); raw `name` avoided.
- [ ] `options_list` used for short flags/overrides; help lives in help files, not inline.
- [ ] Name-or-ID: single `--storage-account`; **no** `--*-id` / `--*-resource-group` siblings.
- [ ] `--ids` not on `create`; suppressed on child `list` (`id_part=None`).
- [ ] Booleans use `get_three_state_flag()` (persisted) or `action='store_true'` (switch); enums use `get_enum_type()`.

### [4. Module structure & authoring](#4-module-structure-and-authoring)
- [ ] Loader extends `AzCommandsLoader`; no heavy work in `__init__`.
- [ ] `commands.py` (table), `_params.py` (arguments), `custom.py` (logic), `_help.py`/`help.yaml` (help).
- [ ] Custom funcs: `cmd` first if used; `client` only with a registered `client_factory`.
- [ ] `--no-wait` via `supports_no_wait=True` + `sdk_no_wait(...)`; `--defer` via `supports_local_cache=True` + `cached_get/put`.

### [5. Complex types & collections](#5-complex-types-and-collections)
- [ ] **No raw JSON-blob arguments**; object props flattened into individual args.
- [ ] Child collections use subcommand groups (`upsert_to_collection`), not argparse Actions.
- [ ] Output mirrors service response; custom table via callable/`table_transformer` only when needed.

### [6. Special command patterns](#6-special-command-patterns)
- [ ] Managed identity uses `--mi-system-assigned`/`--mi-user-assigned` (create) and `identity assign/remove/show`.
- [ ] Private endpoint connection registered correctly; `private-link-resource list` returns an **array**; mandatory integration test present.
- [ ] Network rules use `network-rule add/remove/list` with rule-set props on parent under `Network Rule` group.

### [7. Error handling](#7-error-handling)
- [ ] **No `CLIError`** in new code; raises **third-layer** `azure.cli.core.azclierror` types.
- [ ] No base/second-layer types; specific type chosen over fallback.
- [ ] Messages: capitalized, actionable, suggest the arg; no styling/`\n`, no usage/type prefixes, no regex/vague text.
- [ ] `recommendation` supplied where the fix isn't obvious.

### [8. Help & reference docs](#8-help-and-reference-docs)
- [ ] YAML help; `_help.py` imported in `__init__.py`; param names match CLI output; `` `<angle>` `` quoted; verified with `-h`.
- [ ] short-summary ≤200 chars, single active-voice sentence; long-summary ≤2000 chars; Markdown links w/o locale/monikers.
- [ ] ≥2 examples/command + ≥1/parameter; tested in Bash & PowerShell; no HTML tags.
- [ ] Parameter help documents accepted/default/example values & formats; billable-resource groups have a conceptual article.

### [9. Tests](#9-tests)
- [ ] New module/commands have tests; scenario tests repeatable in live mode.
- [ ] `ScenarioTest`/`LiveScenarioTest`; methods named `test_<module>_<feature>`.
- [ ] No hard-coded resources; `ResourceGroupPreparer`/`StorageAccountPreparer` (RG preparer first); `self.create_random_name(...)`.
- [ ] Recordings committed, secrets scrubbed; assertions via `self.cmd(...)`, `self.check(...)`, `get_output_in_json()`.

### [10. Breaking changes](#10-breaking-changes)
- [ ] Change identified against rule codes 1001–1012 (see table).
- [ ] Inside a Breaking Change Window (~May/Build, ~Nov/Ignite) and pre-announced ≥30 days.
- [ ] Registered in module `_breaking_change.py` (not `deprecate_info`); correct `target_version`.
- [ ] Deprecation not combined with another breaking-change registration on the same item; old+new coexist during pre-announcement.
- [ ] `azdev generate-breaking-change-report` shows it; reviewed by CLI team; merged before code freeze.

### [11. Extensions](#11-extensions)
- [ ] No load-order/inter-extension dependency; new error types need `minCliCoreVersion ≥ 2.15.0`.
- [ ] `azext_metadata.json` correct; **stable release has neither** `isPreview` nor `isExperimental`.
- [ ] Version `MAJOR.MINOR.PATCH[b<n>]`; correct bump (breaking→major, feature→minor, fix→patch).
- [ ] `src/index.json` not hand-edited (hosted); summary ≤3 sentences, ~120–140 chars, specific.

### [12. Track 2 SDK migration](#12-track-2-sdk-migration)
- [ ] LRO methods use `begin_`; errors use `azure.core.exceptions`; enums-as-strings (no `.value`).
- [ ] `get_subscription_id(cmd.cli_ctx)` used; SDK bumped in `setup.py` + `requirements.py3.{Windows,Linux,Darwin}.txt`; multi-API intact; mocks updated.

### [13. Security, local context, telemetry, misc](#13-security-local-context-telemetry-misc)
- [ ] Subprocess via `run_cmd` with arg array; never `shell=True` with user input.
- [ ] `GraphClient` via `graph_client_factory`; no direct instantiation.
- [ ] Local context only on resource-dependency params; no `GET` for `name` in create/delete; `SET` only in `create`.
- [ ] No secrets logged to telemetry; storage data-plane auth flags supported.

### [14. Coding & CI](#14-coding-and-ci)
- [ ] Output objects/dict/`None` only; goes to stdout; logging (not `print()`) for messages.
- [ ] Passes pylint + pep8; commands have tests; supports Python 3.10–3.14.

---

# Part 2 — Detail

## 0. Review workflow and local validation
- Dev tooling is **`azdev`** (`azure-cli-dev-tools`); set up via `azdev setup`.
- Validate a module: `azdev test <module>`, `azdev style <module>`, `azdev linter <module>`.
- Coverage: `azdev cmdcov <module>` (`--level argument` for per-arg coverage).
- **CI gate:** style (pylint + pep8), linter, tests must pass. Code supports **Python 3.10–3.14**. (\*) All commands should have tests.

## 1. PR hygiene
- **Title MUST start** with `[Component]` (customer-facing → written to `HISTORY.rst`) or `{Component}` (not customer-facing → not in HISTORY). Component is title-cased with spaces (`Storage`, `API Management`, `Packaging`, `Misc.`, `Aladdin`).
- `Core` component → `src/azure-cli-core/HISTORY.rst`; otherwise `src/azure-cli/HISTORY.rst`.
- 2nd part: `BREAKING CHANGE:` (breaking), `Hotfix` (hotfix), `Fix #<n>` (issue). Otherwise may be empty.
- Verb (recommended): present-tense base form, capitalized — **Add / Change / Deprecate / Remove / Fix**.
- History notes auto-generate from title/description; use a `History Notes` section for multiple/override notes.
- **Hotfix PRs** branch off `release`; customer-facing hotfixes edit `HISTORY.rst` manually. Merge `release` back to `dev` via a **merge commit — never squash**.

## 2. Command naming and behavior
- Pattern **`[noun] [noun] [verb]`**; every command name contains a verb (`account get-connection-string`, not `account connection-string`).
- Multi-word subgroups hyphenated (`foo-resource`). Avoid hyphenated command names if a subgroup removes the need (`database show`, not `show-database`).
- Avoid empty/single-command subgroups that add tree depth (`keyvault create`, not `keyvault vault create`) — unless siblings are planned soon.
- Child collections: `CREATE`/`DELETE` **or** `ADD`/`REMOVE`.
- (\*) Don't invent single-word verbs that clash with standard types (`get`≈show, `new`≈create). Prefer descriptive hyphenated names.

**Standard command types (MUST follow):**

| Verb | Backing | Rules |
|---|---|---|
| `CREATE` | PUT | idempotent; returns the resource |
| `UPDATE` | PUT/PATCH (PATCH-like) | register via `generic_update_command` (exposes `--set/--add/--remove`); returns updated resource |
| `SET` | PUT | replaces all props (rare); returns resource |
| `SHOW` | GET | register via `show_command`/`custom_show_command` → 404 returns exit code **3** |
| `LIST` | GET | single command; behavior via args (`--resource-group` → list_by_rg else list_by_subscription) |
| `DELETE` | DELETE | returns nothing on success |
| `WAIT` | polls GET | required if any command in the group exposes `--no-wait` |

## 3. Argument conventions
- **No units in names** — put units in help (`--duration`, not `--duration-in-minutes`); accept unit suffixes where possible. Exception: enum-like (`--start-day`, `--start-hour`).
- `-id` suffix ⇒ GUID. **ARM-ID** args omit `-id` and use the **"name or ID"** overload.
- Overload rather than duplicate (`--parameters` accepting path OR URL — not `--parameters-url` + `--parameters-path`).
- Prefer canonical param names: **`resource_group_name`** (gets `-g`, alias, completer), avoid `rg`. **Avoid `name`** as a raw param (alias conflicts).
- snake_case params auto-map to `--kebab-case`; override with `options_list` (str or list, e.g. `['--myparam','-m']`).
- **Name-or-ID:** expose `--storage-account` ("Name or ID of…"); **DO NOT** add `--storage-account-id` or `--storage-account-resource-group`. Resolve in a validator using `is_valid_resource_id`/`resource_id`/`get_subscription_id`.
- `id_part` enables `--ids`; `--ids` is **never exposed on `create`** and should be **suppressed on child `list`** (set parent `id_part=None`).
- Booleans: **`get_three_state_flag()`** for persisted properties; `action='store_true'` only for non-persisted switches. Enums: **`get_enum_type()`** (case-insensitive; preferred over `choices`).
- Group related args with `arg_group=`. Prefer help files over inline `help=`.

## 4. Module structure and authoring
- Loader inherits `AzCommandsLoader`; sets `COMMAND_LOADER_CLS`. No heavy work in `__init__`.
- `commands.py` → `load_command_table`; `_params.py` → `load_arguments`; custom logic in `custom.py`; help in `_help.py`/`help.yaml`.
- Register with `with self.command_group('grp', cli_command_type) as g:` and `g.command / g.custom_command / g.generic_update_command / g.wait_command / g.show_command`.
- Custom funcs are plain functions; `cmd` (if used) is **first param**, `client` injected when a `client_factory` is registered. Missing default ⇒ required.
- `--no-wait`: `supports_no_wait=True`; in custom code use `sdk_no_wait(no_wait, func, ...)`.
- `--defer`: `supports_local_cache=True` + `cached_get`/`cached_put`.
- Arg contexts: `with self.argument_context('scope') as c:` — `c.argument / c.ignore / c.extra`. Rules apply generic→specific.

## 5. Complex types and collections
- **JSON blobs as args are UNACCEPTABLE.** Flatten object props into individual args (assemble in custom code/validator; use `cmd.get_models(...)`).
- Child collections: prefer a **subcommand group** (`foo rule create ...`) with `upsert_to_collection`/`get_property`. Argparse **Actions are NOT RECOMMENDED** (no tab-completion, harder UX, more maintenance).
- Service can't accept an empty collection? Use `--defer` caching (`supports_local_cache`, `cached_get`/`cached_put`).
- Output should mirror service responses; custom table output via callable→`OrderedDict` or `table_transformer` (JMESPath); transformation should be infrequent.

## 6. Special command patterns
- **Managed identity:** create-time `--mi-system-assigned` / `--mi-user-assigned <ids>`; existing resource → `identity` subgroup with `assign`/`remove`/`show` using `--system-assigned`/`--user-assigned`. `remove --user-assigned` (no value) removes all, with warning.
- **Private endpoint connection:** register service into `az network private-endpoint-connection` (`_register_one_provider`); parent exposes `private-endpoint-connection` group (`approve`/`reject`/`delete`/`show`, + `wait` & `--no-wait` if LRO). `private-link-resource` group has only `list` and **must return an array, not a dict**. **Integration test is mandatory** (create → list link resources → create PE → approve → reject → show → delete).
- **Network rules:** single `network-rule` group with `add`/`remove`/`list`; rule-set-wide props (e.g. `default_action`) on parent create/update under a `Network Rule` arg group.

## 7. Error handling
**PRs violating these rules are rejected.**
- **Never `CLIError`** (deprecated). Raise **third-layer** types from `azure.cli.core.azclierror` (`ResourceNotFoundError`, `RequiredArgumentMissingError`, `InvalidArgumentValueError`, `MutuallyExclusiveArgumentError`, …). Never use base (`AzCLIError`) or second layer (`UserFault`/`ClientError`/`ServiceError`). Avoid fallback types when a specific one fits. New general error types inherit a second-layer class (ask AzCLIDev@microsoft.com first).
- Signature: `Error(error_msg, recommendation=None)`; provide actionable `recommendation` (string or list) or call `set_recommendation(...)`.
- **Message DOs:** capitalized; actionable + suggest the arg (`… provide a resource group name by --resource-group`).
- **Message DON'Ts:** no manual styling/`\n`/color; no error-type/usage prefixes; no regex/programming expressions; no vague "Something unexpected happened."

## 8. Help and reference docs
- Author via YAML (`_help.py`/`help.yaml`); `_help.py` must be imported in `__init__.py`. Parameter names in help **must match** CLI output (incl. abbreviations). Quote `` `<angle>` `` to avoid HTML parsing. Verify with `az … -h` (authoring errors only surface at runtime).
- **short-summary:** single sentence, active voice, **≤200 chars**, noun+verb+object.
- **long-summary:** paragraph, tips/expectations/cautions, **≤2000 chars**; Markdown links, strip locale (`/en-us/`) & monikers (add `&preserve-view=true` if a moniker is needed).
- **Examples:** ≥2 per command (one common, one advanced/filter) + ≥1 per parameter; diverse; test in **both Bash and PowerShell** (separate blocks only for real syntax differences); use `\` (Bash) / `` ` `` (PowerShell) line continuations; no HTML tags; use ` ```azurecli `/` ```azurecli-interactive `.
- **Parameter help** describes the param (accepted/default/service-default/example values, pairing rules, exact multi-value format). Conceptual article required for each group that creates a **billable** resource. New top-level group → update `titleMapping.json` / `service_name.json`; renamed TOC node needs a redirect.

## 9. Tests
- New modules and new commands **MUST include tests**. Scenario tests **must run repeatedly in live mode**.
- `ScenarioTest` (VCR record/replay) is default; `LiveScenarioTest` for un-replayable/live-only. Method names `test_<module>_<feature>`.
- Recordings at `recordings/<profile>/<test>.yaml` (created pass or fail). Re-record by deleting + rerun or `azdev test <t> --live` (or env `AZURE_TEST_RUN_LIVE`). Missing recording ⇒ live.
- **No hard-coded/persistent resources.** Use **`ResourceGroupPreparer`** (injects `resource_group`, key `rg`), `StorageAccountPreparer` (place **below** RG preparer; preparers run top-to-bottom). Names via **`self.create_random_name(...)`** with correct length. Credentials scrubbed from recordings.
- Assertions: `self.cmd(...)` asserts exit 0; `.get_output_in_json()`; `self.check('<jmespath>', value)`, `checks=[...]`, `is_empty`; `self.assertRaisesRegexp(<ErrorType>, <regex>)` for errors. Import helpers from top-level `azure.cli.testsdk`.

## 10. Breaking changes
**What's breaking:** renaming params/commands; changing input logic, output format/props, or behavior; adding blocking verification; removing commands/subgroups/params; making a param required; changing defaults.

**Detector rule codes:**

| Code | Meaning | Breaking? |
|---|---|---|
| 1001 Cmd added / 1011 Subgroup added | additive | No |
| 1002 Cmd removed / 1007 Param removed / 1012 Subgroup removed | removal | **Yes** |
| 1003 Cmd property added | only `confirmation` | Conditional |
| 1004 / 1005 Cmd property removed / updated | — | No (empty break lists) |
| 1006 Param added | only if **required** | Conditional |
| 1008 Param property added | `required` or `choices` | Conditional |
| 1009 Param property removed | `options`, `id_part`, `nargs` | Conditional |
| 1010 Param property updated | `default` / `aaz_default` | Conditional |

**Process (reviewer must require):**
- Only in the **bi-annual Breaking Change Window** (~May/Build, ~Nov/Ignite); outside is normally prohibited.
- **Pre-announced ≥30 days** (≈2 sprints for Core) — via a `Breaking Change`-labeled issue (Core-owned modules) or PR (service-owned modules).
- Registered in the module's **`_breaking_change.py`** (preferred over `deprecate_info`): `register_command_deprecate`, `register_argument_deprecate`, `register_command_group_deprecate`, `register_output_breaking_change`, `register_logic_breaking_change`, `register_default_value_breaking_change`, `register_required_flag_breaking_change`, `register_conditional_breaking_change`, `register_other_breaking_change`.
- **Do not** combine a deprecation and another breaking-change registration on the same item. Old + new behavior should coexist during pre-announcement. Use `register_conditional_breaking_change` (not raw `logger.warning`).
- `target_version` defaults to the next window (specific like `2.70.0`, or approximate like `May 2025`). Verify `azdev generate-breaking-change-report` shows the announcement.
- Detector extension: `az extension add --name command-change` → `az command-change version-diff` / `meta-diff --only-break`.
- Must be reviewed by a **CLI team member** and merged before **sprint code freeze**.

## 11. Extensions
- Same authoring as modules but user-installable (Python wheel). Names need **not** start with `azure-cli-`. **Cannot depend on load order** → can't depend on other extensions. New/preview error types require `azext.minCliCoreVersion ≥ 2.15.0` (adopting new error types is optional for extensions).
- `azext_metadata.json`: `azext.minCliCoreVersion`/`maxCliCoreVersion` (inclusive), `azext.isPreview`, `azext.isExperimental`. **Stable releases must have neither** isPreview nor isExperimental.
- **Versioning** `MAJOR.MINOR.PATCH[b<n>]`: stable starts `1.0.0`, preview `1.0.0b1`. Stable: breaking→major, feature→minor, fix→patch (a bump resets lower parts to 0).
- Don't hand-edit `src/index.json` for hosted extensions; release triggers on version change (`python setup.py --version`). Vendor Azure SDKs under `vendored_sdks`; keep deps minimal, don't override core deps.
- **Summary text:** complete sentences, ≤3 sentences, ~120–140 chars, specific, with example commands; don't start with "An extension"/"Azure CLI".
- **Load/override order:** module < wheel extension < dev extension; same level = alphabetical (later wins). A preview overriding GA needs a name sorting **after** GA (e.g. `containerapp-preview`).

## 12. Track 2 SDK migration
Watch for: LRO methods gain **`begin_`** prefix (`begin_create_or_update`); errors move to `azure.core.exceptions` (`CloudError`→`ResourceNotFoundError`, `ClientException`→`HttpResponseError`); enums may become strings — **don't use `.value`**; property/class renames; flattening may disappear (build nested models); subscription via **`get_subscription_id(cmd.cli_ctx)`** (not `client.config.subscription_id`). Bump SDK in `setup.py` + `requirements.py3.{Windows,Linux,Darwin}.txt`; keep multi-API support; update test mocks.

## 13. Security, local context, telemetry, misc
- **Subprocess:** don't use `subprocess.run/Popen/check_call/check_output/call`; use CLI's **`run_cmd`** with an **argument array** (never `shell=True` with user input). Prefer SDK op → atomic AAZ command → `run_az_cmd`. Use `shutil.which`; Windows built-ins need `["cmd.exe","/c",…]`.
- **Microsoft Graph:** never instantiate `GraphClient` directly — use `graph_client_factory`; body/response are `dict`s; errors raise `GraphError`.
- **Local context / param persist** (`LocalContextAttribute`: name / scopes / actions GET|SET): enable only for **resource-dependency** params; **no `GET` for `name` in create/delete**; define `SET` only in `create`.
- **Storage data-plane auth:** commands hit Storage directly — support `--auth-mode login` / `--account-key` / `--connection-string` / `--sas-token`; default queries account key.
- **Telemetry:** command telemetry logged to Kusto `RawEventsAzCli`; `ResultSummary`/`ExceptionMessage` may be suppressed for privacy; extensions add data via `add_extension_event`. Don't log secrets.
- **CLI-team-owned modules** (ownership context): `compute` (vm/vmss/disk/sig/…), `network` (excl. `private-dns`), `resource` (group/deployment/policy/account/…), `role` (ad/identity), `storage`, plus keyvault/monitor/billing/etc.

## 14. Coding and CI
- All command output must be an **object, dict, or `None`** (not str/bool). Output → **stdout**; logs/status/errors → **stderr** via `logger.*` — **do not `print()`**.
- Be POSIX-friendly (pipe/grep/jq); support JSON/TSV/table output (custom table when needed); support tab completion.
- Passes **pylint + pep8**; (\*) commands have tests; supports **Python 3.10–3.14**; PRs to `azure-cli` / `azure-cli-extensions` must pass CI.

---

## Source docs (in `azure-cli/doc/`)
`command_guidelines.md` · `error_handling_guidelines.md` · `authoring_help.md` · `authoring_tests.md` · `reference_doc_guidelines.md` · `authoring_command_modules/{authoring_commands,README}.md` · `how_to_introduce_breaking_changes.md` · `breaking_change_detection_guidelines.md` · `breaking_change_rules/1001–1012.md` · `extensions/{authoring,versioning_guidelines,metadata,extension_summary_guidelines,faq,README}.md` · `managed_identity_command_guideline.md` · `private_endpoint_connection_command_guideline.md` · `local_context_guidelines.md` · `parampersist_localcontext_guidelines.md` · `cli_subprocess_guidelines.md` · `microsoft_graph_client.md` · `storage_data_plane_commands_auth.md` · `track_2_migration_guidance.md` · `version_management.md` · `telemetry/{README,faq}.md` · `modules_owned_by_cli_team.md` · `overwrite_order_in_cli.md` · `quoting-issues-with-powershell.md` · `how_to_bump_SDK_version_in_cli.md` · `configuring_your_machine.md`
