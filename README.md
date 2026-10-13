# JX3Rocks Helm Chart Repository

This repository hosts Helm charts used by JX3Rocks Kubernetes projects, including Jenkins X / JayeX GitOps environments.

The repository is published using **GitHub Pages** and is intended to provide a central location for distributing versioned Helm chart packages.

## 1. Repository Information

| Property | Value |
|---|---|
| GitHub repository | https://github.com/jx3rocks/repo |
| Publishing branch | `gh-pages` |
| Public Helm repository | https://jx3rocks.github.io/repo/ |
| Helm repository index | `index.yaml` |
| Chart package format | `.tgz` |
| Local Helm alias | `myrepo` |
| Repository maintainer | JX3Rocks |

The repository uses a flat structure:

```text
repo/
├── README.md
├── index.yaml
├── chart-name-0.0.1.tgz
├── chart-name-0.0.2.tgz
└── ...
```

Chart packages are stored in the root of the `gh-pages` branch.

GitHub Pages makes these files accessible through the public repository URL.

## 2. Helm Repository Architecture

The publishing process is:

```text
Helm Chart Source
       |
       v
helm package
       |
       v
Chart Archive (.tgz)
       |
       v
GitHub Repository (gh-pages)
       |
       v
index.yaml
       |
       v
GitHub Pages
       |
       v
Helm Clients / GitOps Deployments
```

**Important distinctions:**

- A Helm chart is a package of Kubernetes templates and configuration.
- A Helm repository distributes chart packages and maintains an index of available versions.
- A Helm release represents an installation managed by Helm.
- JayeX GitOps may render and apply charts without registering each application as a conventional Helm release.
- Removing a chart from this repository does not automatically uninstall running Kubernetes resources.
- Removing a chart version still referenced by GitOps may break future deployments or reconciliation.

## 3. Configure the Local Helm Client

Add the repository:

```bash
helm repo add myrepo https://jx3rocks.github.io/repo/
```

Refresh the repository index:

```bash
helm repo update myrepo
```

List configured repositories:

```bash
helm repo list
```

Search available charts:

```bash
helm search repo myrepo/
```

Show every indexed version of a chart:

```bash
helm search repo myrepo/node-https-aks05 --versions
```

Inspect chart metadata:

```bash
helm show chart myrepo/node-https-aks05 --version 0.0.5
```

Inspect chart values:

```bash
helm show values myrepo/node-https-aks05 --version 0.0.5
```

The chart name and version are examples; substitute the chart you want to inspect.

The local Helm alias `myrepo` is independent of the repository aliases configured in GitOps Helmfiles.

## 4. Clone and Maintain the Repository

Clone the GitHub repository:

```bash
git clone https://github.com/jx3rocks/repo.git
cd repo
```

Switch to the publishing branch:

```bash
git checkout gh-pages
```

Update the local checkout:

```bash
git pull --ff-only origin gh-pages
```

Verify the current branch:

```bash
git branch --show-current
```

Expected:

```text
gh-pages
```

Check repository status:

```bash
git status
```

Review recent commits:

```bash
git log -5 --oneline
```

**Always verify that the working directory is clean before beginning a new maintenance operation.**

## 5. Publish a New Helm Chart

The chart source normally exists outside this publishing repository.

For example:

```text
my-application/
├── Chart.yaml
├── values.yaml
└── templates/
```

### Package the chart

From the chart source's parent directory:

```bash
helm lint ./my-application
helm package ./my-application
```

This produces a package similar to:

```text
my-application-0.0.1.tgz
```

The chart name and version come from `Chart.yaml`.

### Copy the package into the publishing repository

```bash
cp my-application-0.0.1.tgz /path/to/repo/
```

Replace `/path/to/repo/` with the local checkout of this repository's `gh-pages` branch.

### Update the Helm index

From the publishing repository root:

```bash
helm repo index . \
  --url https://jx3rocks.github.io/repo/ \
  --merge index.yaml
```

The `--merge` option preserves existing indexed chart versions when publishing a new package.

Use this option for normal publishing, not for repository cleanup.

### Review and publish

```bash
git status
git diff -- index.yaml
git add index.yaml *.tgz
git commit -m "Publish Helm chart"
git push origin gh-pages
```

If an identical package filename already exists, investigate before overwriting it. Published chart versions should be treated as immutable.

## 6. Publish an Updated Chart Version

When modifying an existing chart:

1. Update the chart source.
2. Increment `version` in `Chart.yaml`.
3. Update `appVersion` when the application version changes.
4. Run `helm lint`.
5. Package the updated chart.
6. Publish the new package and update the index.

Example `Chart.yaml`:

```yaml
apiVersion: v2
name: node-https-aks05
description: A Helm chart for Kubernetes
type: application
version: 0.0.6
appVersion: "0.0.6"
```

The versions above are illustrative.

**Do not reuse a previously published chart version for different contents.**

GitOps configurations that pin a chart version must be updated separately when you want them to consume a newer version.

## 7. Inspect Published Charts

List the available chart names and their latest indexed versions:

```bash
helm search repo myrepo/
```

List all versions:

```bash
helm search repo myrepo/ --versions
```

Count chart packages in the Git checkout:

```bash
find . -maxdepth 1 -name '*.tgz' -type f | wc -l
```

Inspect the repository index:

```bash
head -30 index.yaml
```

Inspect the publicly served index:

```bash
curl -fsSL https://jx3rocks.github.io/repo/index.yaml | head -30
```

The Git checkout, public index, and local Helm cache are three different views of the repository.

They may temporarily disagree if changes have not been committed, pushed, published by GitHub Pages, or refreshed locally.

## 8. Remove Obsolete Chart Versions

**Warning:** Before removing published charts, verify that no active GitOps environment requires those chart versions.

First, synchronize the publishing branch and create a recovery tag:

```bash
git checkout gh-pages
git pull --ff-only origin gh-pages
git status --short
```

Proceed only with a clean working tree.

```bash
git tag helm-repo-before-cleanup-YYYYMMDD
git push origin helm-repo-before-cleanup-YYYYMMDD
```

Replace `YYYYMMDD` with the current date and use a unique tag name.

### Remove selected packages

Preview the exact files:

```bash
ls -l chart-name-*.tgz
```

Stage removal of explicitly selected versions:

```bash
git rm chart-name-0.0.1.tgz
```

Repeat only for versions confirmed to be obsolete.

### Rebuild the index

After removing obsolete packages, regenerate the index from the packages remaining in the directory:

```bash
helm repo index . \
  --url https://jx3rocks.github.io/repo/
```

**Do not use `--merge` when removing obsolete chart versions.**

Merging may preserve index entries for packages that have been removed.

Review:

```bash
git add index.yaml
git diff --cached --stat
git diff --cached -- index.yaml
```

After verification:

```bash
git commit -m "Remove obsolete Helm chart versions"
git push origin gh-pages
```

## 9. Reset the Repository to an Empty State

A full reset removes all published chart archives and clears the chart index while preserving the repository and README.

**This is a destructive publishing operation. Confirm that active GitOps configurations no longer depend on the charts before proceeding.**

Create a backup tag as described in Section 8.

Preview the packages:

```bash
find . -maxdepth 1 -type f -name '*.tgz' -print | sort
```

Stage removal of root-level chart archives:

```bash
git rm -- '*.tgz'
```

Replace `index.yaml` with an empty Helm repository index:

```bash
cat > index.yaml <<EOF
apiVersion: v1
entries: {}
generated: "$(date -u +%Y-%m-%dT%H:%M:%SZ)"
EOF
```

Stage the index:

```bash
git add index.yaml
```

Review:

```bash
git status --short
git diff --cached --stat
```

Commit and publish only after confirming the intended changes:

```bash
git commit -m "Reset Helm chart repository to empty"
git push origin gh-pages
```

The repository remains usable for future Helm chart publication.

## 10. Refresh and Verify the Published Repository

After publishing changes, check the remote index:

```bash
curl -fsSL https://jx3rocks.github.io/repo/index.yaml
```

Refresh the local Helm cache:

```bash
helm repo update myrepo
```

Search the repository:

```bash
helm search repo myrepo/
```

An empty repository should produce:

```text
No results found
```

If old charts still appear, check whether:

- The changes were committed and pushed to `gh-pages`.
- GitHub Pages has finished publishing.
- The public `index.yaml` reflects the expected changes.
- The local Helm cache has been refreshed.

Do not assume that `helm search repo` reflects the current Git working directory.

## 11. GitOps Considerations

This repository may be consumed by JayeX / Jenkins X GitOps configurations.

GitOps Helmfiles can specify:

- A Helm repository URL.
- A chart name.
- A pinned chart version.
- Environment-specific values.

Before deleting a published chart version, inspect the relevant GitOps repositories for references to that chart and version.

Example search from the root of a GitOps repository:

```bash
git grep -n 'node-https-aks05'
```

Review all active environment configurations, not just the current Kubernetes context.

A GitOps configuration can reference a chart repository directly without requiring a corresponding alias in the local Helm client.

Deleting a local Helm alias does not delete the published repository.

## 12. Backup and Recovery

Git history preserves committed chart packages and repository index files.

Before major cleanup operations, create a Git tag and push it to GitHub.

To inspect an earlier snapshot:

```bash
git show helm-repo-before-cleanup-YYYYMMDD:index.yaml
```

To restore the repository contents from a backup tag, first ensure the working tree is clean and that the tag contains the intended state.

One recovery approach is to restore tracked content from the tag:

```bash
git restore --source=helm-repo-before-cleanup-YYYYMMDD \
  --staged --worktree -- .
```

This restores files present in the tagged snapshot. Files introduced afterward may require separate review or removal.

Always inspect the resulting changes before committing or pushing.

Do not rewrite or force-push Git history merely to restore an older chart repository state.

## 13. Recommended Maintenance Practices

- Keep the `gh-pages` branch dedicated to published repository content.
- Maintain `README.md` as the repository's operations guide.
- Preserve immutable, versioned `.tgz` chart packages.
- Update `index.yaml` whenever publishing or removing charts.
- Use `--merge` when adding packages and rebuilding with retained history.
- Rebuild without `--merge` when removing obsolete packages.
- Review Git changes before every commit.
- Back up the repository before large cleanup operations.
- Verify GitOps dependencies before removing chart versions.
- Refresh the local Helm cache after publishing changes.
- Periodically inspect the repository for obsolete packages and unused chart versions.

## 14. Quick Reference

| Task | Command |
|---|---|
| List local Helm repositories | `helm repo list` |
| Refresh repository | `helm repo update myrepo` |
| Search published charts | `helm search repo myrepo/` |
| Show all chart versions | `helm search repo myrepo/ --versions` |
| Inspect chart metadata | `helm show chart myrepo/CHART --version VERSION` |
| Inspect chart values | `helm show values myrepo/CHART --version VERSION` |
| Package chart | `helm package ./CHART_DIRECTORY` |
| Rebuild index | `helm repo index . --url https://jx3rocks.github.io/repo/` |
| Check Git changes | `git status` |
| Inspect published index | `curl -fsSL https://jx3rocks.github.io/repo/index.yaml` |

---

**Repository:** https://github.com/jx3rocks/repo

**Published Helm endpoint:** https://jx3rocks.github.io/repo/

**Publishing branch:** `gh-pages`
