How to do a new release
=======================
Releases are cut from master. The `Create signed release tag` workflow signs
the tag and starts the `Release to PyPI` workflow, which tests, builds and
publishes the release. Building rpm or deb packages is not part of it.

Before the first release
------------------------
A repository admin and a PyPI owner set up, once:
- the `pypi` environment (Settings > Environments), with the release managers
  as required reviewers and deployments limited to `spec-cleaner-*` tags,
- a GitHub trusted publisher on PyPI for this repository, the workflow
  `release.yml` and the environment `pypi`, removing any publisher or API
  token that is no longer used,
- the signing subkey secrets that `create-release-tag.yml` reads, and the
  matching public key, signing subkey included, on the GitHub account whose
  key `release.yml` imports to verify the tag.

Each release manager needs write access to run the tagging workflow and must
be a reviewer of the `pypi` environment.

Versions
--------
Bump the minor version when the default output, the command line options or
the supported Python versions change, and the patch version for fixes and data
refreshes. After a release, master carries the next patch version.

Steps
-----
1. Run `make` on master. If it changes anything, merge that refresh first, as
   the release checks that `make` leaves no diff.
2. Check that `spec_cleaner/__init__.py` has the version to release, and bump
   it in a pull request if not.
3. Run the `Create signed release tag` workflow on master (Actions tab, or
   `gh workflow run create-release-tag.yml --ref master`). It tags
   `spec-cleaner-X.Y.Z` and starts `Release to PyPI` for it.
4. The release workflow runs the tests, verifies the tag signature, checks
   that the tag matches `spec_cleaner.__version__` and that `make` leaves no
   diff, and builds the sdist and wheel.
5. Approve the `pypi` deployment in the workflow run. The sdist and wheel are
   uploaded to PyPI, and a GitHub release is created with generated notes and
   the same files attached. Optionally add a short summary above the notes.
6. Post release version bump in `spec_cleaner/__init__.py`.

Checking a release
------------------
- The files on https://pypi.org/project/spec-cleaner/ show their provenance.
- `uvx --from spec-cleaner==X.Y.Z spec-cleaner --version` prints X.Y.Z.

When it fails
-------------
- Before the upload to PyPI nothing is published. Fix the problem, delete the
  tag with `git push origin :refs/tags/spec-cleaner-X.Y.Z` (and `git tag -d
  spec-cleaner-X.Y.Z` in any local clone that fetched it), and run the tagging
  workflow again.
- If only the GitHub release failed, re-run the failed job.
- A version on PyPI can never be uploaded again. If it is broken, yank it on
  PyPI and release the next patch version.
