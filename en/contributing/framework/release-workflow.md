# Release Workflow

## Development

Once pull-requests have been merged into `develop` a new release is automatically generated and published to our [MyGet feed](pre-release-builds.md). There is nothing you need to do as the package identifier and library version is [automatically incremented](semantic-versioning.md) based upon our [GitVersion configuration](https://github.com/reactiveui/ReactiveUI/blob/develop/GitVersion.yml).

![commits to develop are automatically pushed to MyGet](/images/contributing/commits-to-develop-are-automatically-pushed-to-myget.png)

## Production

By design, no single person can release a new version of ReactiveUI without approval from [one of the other contributors](https://github.com/orgs/reactiveui/teams/contributors). The process is kicked off by one of the contributors opening up a pull-request from `develop` to `master`

![](/images/contributing/create-a-pull-request-from-develop-to-master.png)

At this stage the contributor that is proposing a new release must perform a few administrative tasks. It's the responsibility of the release approver to verify that these tasks have been performed correctly. Both parties should [jump into the ReactiveUI video chat room](https://appear.in/reactiveui) before proceeding any further.

![](/images/contributing/pull-request-review-required.png)

Edit the `vNext` milestone and change it to the version number of the next release. If this release contains a [breaking change](semantic-versioning.md) then increase the `BREAKING` version by one and reset the `MINOR` back to zero and keep the `PATCH` at zero. Otherwise just bump the `MINOR` version by one and keep the `PATCH` at zero.

![](/images/contributing/click-edit-vnext-milestone-button.png)

If process for [merging individual pull-requests](merging-pull-requests.md) was followed perfectly then you won't need to do much here but in both parties need to verify that all issues that have been assigned to a milestone:

* Has a prefix which classifies the type of change and clearly explained what was changed because our release notes are automatically generated from this information.
* Have _one_ label assigned \(no, more until GitReleaseManager has been PR'ed to support multiple labels\) to the GitHub issue which classifies the scope of change. GitReleaseManager will fail to automatically generate the release notes and the draft release in GitHub if any issue has no labels. If this happens, resolve this in GitHub and then re-run the merge commit build in AppVeyor.

![](/images/contributing/ensure-all-issues-assigned-to-a-milestone-are-labeled.png)

Once the release has been approved, you'll need to switch your merge mode to `Create a merge commit` aka `Merge pull request` by using the little arrow on the right hand side.

![](/images/contributing/merge-commit-option.png)

Verify again that the GitHub milestone has been renamed from **vNext** to the appropriate version, as that will be the version of the software release.

When you merge, be sure to include a message that will cause the semver to be bumped appropriately. One of:

* +semver: breaking
* +semver: feature

![](/images/contributing/merge-commit.png)

Merging this pull-request into `master` will kick off a new build on AppVeyor but the build pipeline is configured not to publish packages to NuGet.org unless the release has been tagged.

![](/images/contributing/commits-to-master-do-not-automatically-publish-to-nuget.png)

Once this build completes, a new draft release with automatically generated release notes will be available for you to polish and add any flair as needed.

![](/images/contributing/edit-release-notes.png)

Pressing the publish release button will stamp the repository with a git tag which the release version, overriding any automatic versioning strategies and trigger a new build which will be automatically published to NuGet.org.

![](/images/contributing/stamp-repository-and-publish-release.png)

![](/images/contributing/pull-request-into-master-then-publish-tag-to-release.png)

![](/images/contributing/tagged-releases-automatically-publish-to-nuget.png)

One final step remains, create a new `vNext` milestone.

![](/images/contributing/create-new-vnext-milestone.png)

### Troubleshooting

#### Version wasn't bumped when merging from develop into master

You'll need to do a pull-request similar to this [https://github.com/reactiveui/ReactiveUI/pull/1226](https://github.com/reactiveui/ReactiveUI/pull/1226)

![](/images/contributing/release-failed-because-gitreleasemanager-could-not-find-the-milestone.png)

Change to a clean copy of `master`

```shell
git clean -fdx
git reset --hard
git checkout master
git pull
```

Create a new branch

```shell
git branch bump-release-version
```

If you want to bump the `BREAKING` create an empty commit in the branch with the following commit message

```
git commit --allow-empty -m "fix: the previous commit didn't bump with +semver: breaking"
```

If you want to bump the `FEATURE` create an empty commit in the branch with the following commit message

```
git commit --allow-empty -m "fix: the previous commit didn't bump with +semver: feature"
```

Push your branch

```shell
git push origin bump-release-version
```

Open a pull-request to `master` and once the release has been approved, you'll need to switch your merge mode to `Create a merge commit` aka `Merge pull request` by using the little arrow on the right hand side.

![](/images/contributing/merge-commit-option.png)

Do not customize the merge commit message or more specifically, do not bump the semver in the merge commit message.

#### Release failed because of labeling issue

Visit the issue, resolve the problem and then visit AppVeyor and click "Rebuild Commit". It is safe to do this multiple times because building `master` does not automatically release unless the commit has been tagged.

![](/images/contributing/release-failed-because-of-labeling-issue.png)

