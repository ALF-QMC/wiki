# Release Process

How ALF releases are managed.

## Release Cycle

- **Minor releases** in **April** and **October** each year
- **Major releases** approximately every 3–4 years, accompanied by a documentation paper

## Versioning

ALF uses semantic-style versioning (`MAJOR.MINOR`). Major bumps indicate significant API or algorithmic changes; minor bumps cover new features, model additions, and improvements. Version metadata is tracked in:

- `CITATION.cff` — structured citation metadata
- `CHANGELOG.md` — detailed log of changes per version
- Documentation LaTeX variable `\ALFver`

## Release Checklist

1. **Tests pass.** Ensure all CI pipelines (test-branch, test-vs-ed) are green across all supported environments (GNU, Intel, PGI, macOS).
2. **Documentation up to date.** Check that the LaTeX documentation paper reflects any new features or API changes.
3. **CHANGELOG.md up to date.** All merged changes since the last release should be listed with author and merge request reference.
4. **Create release branch.** Branch `ALF-X.Y` from `master`.
5. **Update version strings.** Set `\ALFver` in the documentation, update `CITATION.cff`.
6. **Update README.** Add the version number, remove any development warning.
7. **Update master.** Add a development warning to `master`'s README pointing to the release branch.
8. **Companion repos.** Create corresponding branches in [pyALF](https://github.com/ALF-QMC/pyALF) and the Tutorial repository.
9. **Announce.** Update the ALF website and post to Discord.

## CI / CD

Two GitHub Actions workflows support the release process (both `workflow_dispatch`):

| Workflow | Purpose |
|----------|---------|
| `test-branch.yaml` | Compare a feature branch against master using `alf_test_branch`; matrix across Debian (GNU), Intel, PGI, macOS |
| `test-vs-ed.yaml` | QMC vs exact diagonalization regression; prepare → simulate → analyze pipeline |

Both use Docker containers from `ghcr.io/alf-qmc/alf-container/` for reproducible environments.
