# BuddyPress Contributor and Agent Handbook

This guide applies to the BuddyPress development repository. It brings
project safety and review rules together with BuddyPress component, test, and
release guidance. Use the linked contributor and release guides for procedures
that change with the project. Follow the target branch and current repository
configuration when they differ from examples here, and report the discrepancy.

Verify current repository state and project practice before acting. Source,
release branches, supported versions, tests, and release practice can change
independently of this file.

## Mission

Make the smallest correct change that improves BuddyPress while preserving its
public APIs, hooks, component behavior, privacy rules, multisite behavior,
template compatibility, and upgrade path.

Prefer evidence from canonical source, current tests, and project documentation
over assumptions. Leave the working tree reviewable and report what was
verified, what was not, and why.

## Repository Authority

| Purpose | Authoritative location |
| --- | --- |
| Development source | `https://buddypress.svn.wordpress.org/` |
| Public issue tracking | `https://buddypress.trac.wordpress.org/` |
| Git mirror and review surface | `https://github.com/buddypress/buddypress` |
| WordPress.org deployment source | `https://plugins.svn.wordpress.org/buddypress/` |
| Published plugin | `https://wordpress.org/plugins/buddypress/` |
| Contributor documentation | `docs/contributor/` in this repository |
| Confidential security reports | WordPress HackerOne program |

Canonical development happens in Subversion. Its `trunk`, maintenance
branches, tags, revisions, and log determine what shipped. Git mirror commits
and pull requests are review artifacts, not canonical release changes.
Do not merge pull requests or push upstream GitHub mirror branches or tags.
Adding or enabling an automated mirror sync requires separate review and
authorization. Keep project decisions and ticket history on
[BuddyPress Trac](https://buddypress.trac.wordpress.org/).
For a contribution, attach a patch to its Trac ticket or link a GitHub pull
request for code review, as described in the
[pull request template](.github/pull_request_template.md) and
[contribution guide](docs/contributor/code/README.md).

The WordPress.org plugin repository contains generated deployment packages.
Do not edit it as though it were development source. A release build is not a
one-to-one copy of this working tree.

## Non-Negotiable Safety Rules

- Inspect `git status` or `svn status` before and after work. Preserve unrelated
  edits, untracked files, local configuration, and private artifacts.
- Do not use destructive cleanup to obtain a convenient working tree.
- Do not stage, commit, push, open or update a pull request, upload a patch,
  mutate Trac, tag, deploy, publish, or disclose private information without
  explicit authorization for that action and destination. Local editing does
  not authorize publication.
- Never edit an existing tag. Treat tags as immutable release records.
- Do not hand edit generated files when their source or build task is available.
- Keep credentials, tokens, private report identifiers, reproductions, and
  embargoed patches out of source, command arguments, logs, and public services.
- Respect local environment and permission limits. Do not change services,
  install dependencies, or reset databases without approval. Never use a live
  site's database for tests.
- A nonce establishes intent. It does not replace capability, ownership,
  membership, visibility, or object-level authorization checks.
- Confirm the current release process with the BuddyPress team before a public
  commit, tag, deployment, release, or disclosure. A checked-in checklist can
  lag current practice.

## Working Tree Map

- `src/` contains the runtime plugin source.
- Root `bp-loader.php` is the development loader used from this checkout.
- `src/bp-loader.php` is the plugin loader included in release output.
- `src/class-buddypress.php` owns core runtime version and component setup.
- `src/bp-*/` contains component code. Components can be active or inactive and
  can have different visibility and directory behavior.
- `src/bp-templates/bp-legacy/` and `src/bp-templates/bp-nouveau/` contain the
  maintained template packs.
- `src/bp-core/admin/` contains administration screens and modern admin assets.
- `src/js/` and component asset directories contain JavaScript source.
- `tests/phpunit/` contains the PHP test suite and bootstrap.
- `docs/` contains contributor and user documentation.
- `.github/workflows/` describes current CI and the compatibility matrix.
- `Gruntfile.js` defines legacy asset, translation, build, and release tasks.
- `package.json` defines Node requirements, block builds, admin builds, linting,
  and `wp-env` test commands.
- `composer.json` defines PHP dependencies, tests, coding standards, and
  compatibility checks.
- `build/` is generated release output and is ignored. It is not source.

The root loader uses `build/bp-loader.php` when that file exists unless
`BP_LOAD_SOURCE` is defined. Check which tree is loaded before debugging a
source change. Use `BP_LOAD_SOURCE` in an approved development environment
rather than deleting an unrelated build or changing managed configuration.
Some assets inside `src/` are generated too; edit their JavaScript or Sass
inputs and inspect output from the matching build task.

Read the nearest component code and tests before changing behavior. BuddyPress
often exposes the same object through templates, AJAX, REST, feeds, blocks,
emails, notifications, caches, and multisite contexts.

## Choose the Correct Target

`trunk` is the development line for the next release; the Git mirror calls
this branch `master`. Supported maintenance branches contain fixes for
released major lines. Tags are immutable release snapshots.

Before editing:

1. Record the exact revision and working-copy state.
2. Identify the Trac ticket, private report, or maintainer request governing
   the change.
3. Confirm the intended target and affected released branches.
4. Check canonical Subversion, the [Trac roadmap](https://buddypress.trac.wordpress.org/roadmap),
   and the current release plan instead of inferring support from the Git mirror
   or a local branch name.
5. Inspect the same code on every proposed target. Do not assume a clean
   cherry-pick or that a trunk fix covers older releases.

[REST API v2](docs/developer/execution-contexts/rest-api/README.md) arrived in
BuddyPress 15.0.0 and deprecated v1. The separate, archived
[BP-REST repository](https://github.com/buddypress/BP-REST) provides the v1
plugin. For REST changes and older-release backports, inspect the exact target
branch, its Subversion externals, and package ownership before deciding where
the fix belongs. A v2 fix does not establish that v1 is fixed.

## Standard Change Workflow

1. Read the relevant Trac ticket, source, tests, documentation, and history.
2. Reproduce the current behavior with the smallest safe case.
3. Identify the public contract: functions, classes, hooks, filters, return
   values, templates, REST schemas, cache keys, database state, and upgrades.
4. Check component activation, privacy, authentication, role and membership
   boundaries, multisite mode, and template-pack effects as applicable.
5. Implement a narrow source change. Avoid opportunistic refactoring and
   unrelated formatting.
6. Add focused regression coverage for the failure and a legitimate control
   case where practical.
7. Run the narrowest useful check first, then the relevant suite and
   compatibility checks.
8. Inspect the exact diff, generated output, and working-tree status.
9. Prepare a Subversion-compatible patch or a review pull request linked from
   the Trac ticket when one exists and the contribution is ready and authorized.
10. Report the target revision, behavior, compatibility impact, checks, skipped
   checks, unresolved risks, and remaining local changes.

If expected behavior is unclear, stop at diagnosis and present the concrete
choices and compatibility costs before changing the contract.

## Setup and Everyday Commands

Use the versions declared by `package.json`, `composer.json`, and CI. Read
those files when choosing a local runner or a compatibility target; do not
rely on copied version numbers in this guide.

After the environment and dependency changes are authorized, install the
repository's dependencies:

```bash
npm ci
composer install
```

Start and stop the default local WordPress environment when it belongs to this
task:

```bash
npm run wp-env start
npm run wp-env stop
```

Run the primary PHP suites:

```bash
npm run test-php
npm run test-php-multisite
composer test
composer test_multi
```

Run coding and compatibility checks:

```bash
composer phpcs
composer phpcompat
```

The current `composer.json` has no `phpcs-escape` script even though
`Gruntfile.js` still calls it from `grunt commit` and `grunt build`.
Treat that as a workflow failure to investigate and correct in a separately
reviewed tooling change. Do not claim a release build passed by skipping it.

Run the relevant JavaScript lint or build command from `package.json` for files
you change. `npx grunt jstest` runs the JavaScript validation and JSHint tasks
defined in `Gruntfile.js`; block and admin sources have separate scripts.
Do not run a release build merely to hide a source/build mismatch.

PHPUnit accepts `WP_DEVELOP_DIR` or `WP_TESTS_DIR` as described in
`tests/phpunit/includes/define-constants.php`. The npm PHP wrappers assume the
plugin directory is named `buddypress`. A differently named checkout can be
mounted under a different `wp-content/plugins/` directory, so confirm the
`wp-env` plugin path and pass the correct `--env-cwd` when needed.

A missing, inactive, or mis-mounted test environment is not a passing test.
Confirm that WordPress starts and BuddyPress is active before treating results
as a baseline. Because `composer.lock` is ignored, record dependency versions
when exact reproducibility matters.

## Test Expectations

Match coverage to the behavior changed. Start with a focused test, then run the
relevant broader suite.

- Test an unauthorized case and an authorized control case for access changes.
- Test single-site and multisite behavior when the feature supports both.
- Test component-active and component-inactive behavior across components.
- Test public, private, and hidden states for visibility-aware objects.
- Test REST, template, AJAX, feed, block, email, notification, and cache paths
  that expose or transform the same data.
- Test both BP Nouveau and BP Legacy when template behavior changes.
- Test persistent object-cache behavior for cache keys, invalidation,
  visibility, counts, or query results.
- Preserve valid behavior on minimum supported WordPress and PHP versions.
- Use `.github/workflows/` as the source for current CI targets, including
  single-site, multisite, Redis, and Memcached variants where applicable.

The [unit-test workflow](.github/workflows/unit-test-object.yml) and its
reusable workflows define the current PHP, WordPress, multisite, and object
cache matrix. The [coding-standards workflow](.github/workflows/coding-standards.yml)
defines current static checks. Experimental jobs may be advisory only where
the workflow says so; inspect and report their failures. Never weaken or
silence a failed check just to obtain green CI.

Do not call a newly observed failure a regression without comparing it with the
same environment and unmodified source. Report intentional skips and incomplete
coverage precisely.

## Coding and Compatibility Guidance

- Follow WordPress standards and established BuddyPress conventions nearby.
- Use established `bp_` and `BP_` naming patterns for public symbols.
- Preserve public hooks, filters, argument order, return shapes, error codes,
  REST schemas, template variables, and extension points unless the ticket
  deliberately changes them.
- Avoid changing component defaults or directory behavior as a side effect.
- Treat identity, friendship, membership, moderation, and group roles as
  distinct authorization inputs. Do not infer one from another.
- Apply visibility rules before caching or returning data. Ensure a cache cannot
  reuse privileged results in a less privileged context.
- Use BuddyPress APIs for objects, metadata, notifications, activity, and cache
  invalidation when they provide the required behavior.
- Preserve multisite and multiblog behavior, including root-blog assumptions,
  network activation, and global tables.
- Keep BP Nouveau and BP Legacy working unless scope is explicitly narrower.
- Escape on output and sanitize on input at the correct boundary.
- Keep formatting such as `\n` and `\r\n` outside translated strings;
  translate natural-language lines and assemble their plain-text formatting
  in PHP.
- Avoid syntax or APIs outside declared minimum PHP and WordPress versions.
- Do not hide behavioral changes inside broad cleanup or reformatting.

## Version Metadata

Release work can require coordinated updates in:

- Plugin headers in root `bp-loader.php` and `src/bp-loader.php`.
- Runtime version in `src/class-buddypress.php`.
- `package.json` and `npm-shrinkwrap.json`.
- `src/readme.txt`, including `Stable tag`, `Tested up to`, upgrade notice, and
  changelog entries when required by the release type.

Search for the previous version before completing a bump. Not every occurrence
should change. Compatibility comparisons, migration tests, historical docs,
and changelog text can intentionally retain older versions.

Plugin headers, readme requirements, Composer constraints, package requirements,
and CI must agree. Surface inconsistencies instead of silently choosing one.
Update version metadata only for a release or an explicitly scoped version
change, not as incidental cleanup.

## Changelog and Documentation Policy

BuddyPress release notes live primarily in `src/readme.txt`, release pages, and
project announcements rather than a root `CHANGELOG.md`. Follow the current
release checklist for the release type.

Update documentation when a change affects setup, public behavior, extension
APIs, release procedure, or support expectations. Keep historical documentation
accurate. Do not rewrite an old release record to describe current behavior.

Public prose should explain the defect or improvement, technical reason,
compatibility impact, and material risk. Do not include private details or raw
test-count boilerplate.

## Assets, Translation Template, and Release Build

Treat source assets and generated output separately. Use scripts in
`package.json` and `Gruntfile.js` instead of reproducing output manually.

Relevant Grunt tasks include:

- `grunt src` for a development-source build.
- `grunt style` for stylesheets.
- `grunt makepot` for the translation template.
- `grunt commit` and `grunt commit:blocks` for generated commit assets.
- `grunt build` for release output in `build/`.
- `grunt release` for the defined release task sequence.

The release build copies and transforms source, builds block and admin assets,
minifies files, generates translation material, and downloads packaged tools
such as BuddyPress WP-CLI support. Inspect task definitions and prerequisites
before running it. Review output for missing files, stale assets, unexpected
source changes, dependency drift, and strings missing from the translation
template.

Do not add `build/` to development commits unless the release workflow requires
it in the target repository. Validate the generated package instead of assuming
a successful command produced the correct WordPress.org artifact.

## Canonical Subversion Commits

Before an authorized canonical commit:

1. Update the intended Subversion working copy and confirm its URL and revision.
2. Inspect `svn status`, including unversioned and missing paths.
3. Inspect the exact diff and verify every path belongs to the ticket.
4. Run the relevant focused and broader checks.
5. Prepare a project-style message from recent BuddyPress history.
6. Verify props, ticket reference, target branch, and release line.
7. Present the exact diff, target, and proposed message for maintainer review
   before committing.

Recent BuddyPress messages generally lead with a component or concise scope,
use an imperative summary with a period, explain the change when useful, and
end with ticket and release metadata such as:

```text
Component: Correct the specific behavior.

Explain why the change is needed and any compatibility effect.

Props contributor.
Fixes #1234.
For 15.0.
```

Use `See #1234.` when the commit does not close the ticket. Follow current
history for branch wording. Credit humans by their established handles.

Use a message file or another method that stores real line breaks in a
multiline log. After an authorized commit, verify the canonical Subversion log,
revision, diff, and clean working-copy state. Then verify mirror propagation
when it is part of the requested workflow. A Git mirror commit alone is not
completion of a canonical BuddyPress commit.

## Release Procedure

Read `docs/contributor/project/release/build-checklist.md` and
`docs/contributor/project/release/test-checklist.md` in full before release
work. Confirm unclear or stale steps with the BuddyPress team.

The documented flow includes:

1. Prepare release pages, contributor credits, announcements, and Trac
   milestones as required by the release type.
2. Make coordinated version changes in the development repository.
3. Review the exact canonical revision, version metadata, release line, tag
   target, and log before creating an immutable source tag. Commit and tag
   from `trunk` or the correct maintenance branch according to beta, release
   candidate, major, or minor release rules.
4. Generate `build/` from the reviewed, tagged development source. Verify the
   package manifest, translation template, version metadata, and ZIP.
5. Run the release test checklist on fresh single-site and multisite installs,
   component configurations, REST behavior, BP Nouveau, and BP Legacy.
6. After explicit deployment approval, copy only the reviewed distribution to
   the correct WordPress.org plugin repository location, manage its tag, and
   preserve the required stable tag.
7. Verify the published version, then compare the public downloadable ZIP and
   its contents with the reviewed build output.
8. Publish approved announcements and complete Trac milestone cleanup.
9. Perform documented post-release changes on development lines.

Older-line security releases may require a WordPress.org plugin branch without
changing the current stable tag. Source tagging, deployment tagging, public
ZIP propagation, mirror synchronization, and announcements are separate
states. Verify each one; never infer completion from a green build or a source
commit. Never improvise this sequence from memory.

## Security Work

Keep security work private until coordinated disclosure is approved.

1. Privately record the report identifier, reporter claim, affected versions,
   exact revision, required privileges, and affected configurations.
2. Use a separate checkout or worktree with a neutral name when practical.
3. Reproduce the smallest safe case and distinguish confirmed impact, possible
   variants, and ordinary hardening.
4. Trace relevant templates, AJAX, REST, feeds, blocks, emails, notifications,
   caches, CLI, and multisite paths.
5. Prepare a private technical summary: confirmed behavior, approaches,
   compatibility cost, test plan, affected releases, and open questions.
6. Implement the agreed narrow fix and regression coverage. Include an
   unauthorized case and a legitimate control case.
7. Review the final diff and evidence with the authorized BuddyPress team before
   any commit, deployment, message, or disclosure.

For a multi-branch fix, prepare one exact patch artifact per target branch.
Verify each patch's revision and file list. Exclude local documentation,
test-environment configuration, private notes, and unrelated changes. Do not
assume identical code or package ownership across releases, especially for REST.

The `#security-buddypress` WordPress Slack room is a coordination venue. A Slack
discussion does not settle a technical or release decision. Record the agreed
outcome with the report and revision in approved private case notes. Never copy
embargoed material into a public ticket, repository, or build log.

## Documentation and Public Sites

Treat Trac, GitHub, BuddyPress.org, WordPress.org, the developer blog, support
forums, Slack, and HackerOne as separate destinations with different audiences
and confidentiality boundaries.

Before writing publicly, confirm the destination and content are authorized.
Link public work to its Trac ticket. Keep private reproductions and unpublished
patches out of public discussions.

Use repository documentation for durable contributor guidance. Use release
pages and announcements for release-specific communication. Verify live public
results after an authorized publish or deployment.

## Completion Standard

A change is ready for review when:

- The correct canonical target and source revision are identified.
- The confirmed behavior and intended result are clear.
- The diff is narrow and contains no unrelated or private material.
- Relevant WordPress, PHP, component, privacy, template, multisite, REST, and
  cache contexts have been considered.
- Focused coverage and broader checks have run, or each missing check is
  reported with its reason.
- Generated output has been reviewed when build inputs changed.
- Documentation and version metadata are updated where required.
- Final working-tree status is known.
- Remaining risks, branches, release steps, and authorization gates are stated.

Report the outcome first. Include the source revision, affected branches,
confirmed behavior, change and compatibility impact, exact checks and results,
unresolved risks, and working-tree status. Keep confidential details in the
approved private channel.

For a documentation-only change, verify paths and claims, check whitespace
with `git diff --check`, and include untracked files in the review. When
repository behavior or workflows change, update the relevant guidance in a
separately reviewable documentation change. Prefer links to maintained
procedures over copying a version matrix or release checklist here.
