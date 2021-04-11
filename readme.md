# Restore Bitbucket Pipelines Caches

## Problem

[Bitbucket pipelines caches](https://support.atlassian.com/bitbucket-cloud/docs/cache-dependencies/) provide a great way to speed up CI builds, but expire after one week. This can sometimes lead to failed CI work within one's pipeline due to missing dependencies.

## Solution

Leverage Atlassian's existing toolset! They offer a [custom pipe for clearing caches](https://bitbucket.org/atlassian/bitbucket-clear-cache/src/master/), which can be combined in a custom pipeline with other steps to restore your caches during natural development pauses (if you believe in weekends) using [scheduled runs](https://confluence.atlassian.com/bitbucket/scheduled-builds-for-pipelines-933078702.html).

### Steps

1. Create an [App Password](https://support.atlassian.com/bitbucket-cloud/docs/app-passwords/) for the bitbucket-clear-cache pipe to use within your workspace.
  1. Per docs from the custom pipe (classic RTFM) "when generating the app password, remember to check the Pipelines Write and Repositories Read permissions."
  1. Save the password in a secure location! Upon being generated, it will only be shown this once.
1. Test that App Password can retrieve workspace information.
  1. From command line, MacOS/Unix: `curl -u "{bitbucket-username}:{app-password}" "https://api.bitbucket.org/2.0/repositories/{your-workspace-name}"`
  1. One gotcha here - you should be using just your _bitbucket username_, not your login email. This can be found within your [Bitbucket Account Settings](https://bitbucket.org/account/settings/).
  1. Workspace (and repository) names can generally be found within the URL slug of a given Workspace, when you are navigating through the Bitbucket web GUI.
1. Add `BITBUCKET_USERNAME` and `BITBUCKET_APP_PASSWORD` to your [Repository Variables](https://support.atlassian.com/bitbucket-cloud/docs/variables-and-secrets/).
  1. Repository > Repository Settings >  Repository Variables.
1. Implement relevant caches and job definition(s) within your bitbucket-pipelines YAML file.
  1. [Caches are defined and listed](./bitbucket-pipelines.yml#L4) as part of the overall definitions section of your YAML file - an alias with a given subdirectory.
  1. Reference these caches within the relevant job definitions.
    1. [clear-bitbucket-caches](./bitbucket-pipelines.yml#L8) - custom job definition leveraging the Atlassian pipe above.
    1. download-cache-{some-identifier} - in this example, a Node.js application (i.e. [client](./bitbucket-pipelines.yml#L17) or [server](./bitbucket-pipelines.yml#L25)) within a monorepo.
1. Implement custom pipeline(s) definitions.
  1. Thinking about what we have setup, we actually are serving three use cases with our fresh custom pipelines:
    1. [Clearing our caches](./bitbucket-pipelines.yml#L35) - in the case where we want to separate out our scorched-earth approach to removing cached dependencies.
    1. [Downloading our caches](./bitbucket-pipelines.yml#L38) - having a parallel run to download these N number of caches.
    1. [Restoring our caches](./bitbucket-pipelines.yml#L44) - our explicit use case described above, which we'll invoke on a non-intrusive timeline.
  1. Note here that we are leveraging [parallel steps](https://bitbucket.org/blog/speed-build-parallel-steps-pipelines) to save ourselves some time when running these pipelines.
1. [Schedule our Restore Caches pipeline to run](https://confluence.atlassian.com/bitbucket/scheduled-builds-for-pipelines-933078702.html) at a non-intrusive hour (for me, that's ~4AM on a Sunday).
  1. Considering your team's habits and/or timezone, you may want to socialize this before making a heavy-handed (or frustrating) decision.


And that's it! Happy coding, without worrying about cached dependencies being what crashes your CI builds.