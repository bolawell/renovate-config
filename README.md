# renovate-config

I want to manage multiple repositories using Renovate and want the same custom
config across all or most of them, hence I have a preset config so that we can
"extend" it in every applicable repository. This way when I want to change the
Renovate configuration I can make the change in one location rather than having
to copy/paste it to every repository individually.

The [renovate.json](./renovate.json) file in this repository is the preset
config I use in all repositories.

## Preset config

Renovate will simply look for a `renovate.json` file in the default branch,
(for now only the master branch is supported). The renovate bot is configured
to onboard the new repositories with a minimal configuration which will
reference the preset config from this repository.

In other repos, your `renovate.json` should look like:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>eana/renovate-config"]
}
```

## Custom Manager Support using Regex

The `regex manager` is designed to allow users to manually configure Renovate
for how to find dependencies that aren't detected by the built-in package
managers.

The `regex manager` is configured to parse version environment variables within
the `Dockerfile`.

```json
{
  "regexManagers": [
    {
      "fileMatch": ["(^|/)Dockerfile$"],
      "matchStrings": [
        "#\\s*renovate:\\s*datasource=(?<datasource>.*?) depName=(?<depName>.*?)( versioning=(?<versioning>.*?))?\\sENV .*?_VERSION=\"?(?<currentValue>.*?)\"?\\s"
      ],
      "versioningTemplate": "{{#if versioning}}{{versioning}}{{else}}semver{{/if}}"
    }
  ]
}
```

Add environment variables to `Dockerfile` which contain the package version and
annotate them using comments, which the regex manager will end up parsing.

Consider the following `Dockerfile`

```
FROM debian:11.0-slim

# renovate: datasource=github-releases depName=spaam/svtplay-dl versioning=loose
ENV SVTPLAY_VERSION="4.5"

# Install svtplay-dl
RUN set -xe && \
    apt-get update && \
    apt-get install -y --no-install-recommends gnupg curl ca-certificates && \
    curl -s https://svtplay-dl.se/release-key.txt | apt-key add - && \
    echo "deb https://apt.svtplay-dl.se/ svtplay-dl release" | tee /etc/apt/sources.list.d/svtplay-dl.list && \
    apt-get update && \
    apt-get install svtplay-dl=${SVTPLAY_VERSION} -V
```

The example above will look for the newest version of `svtplay-dl` within the
[svtplay-dl](https://github.com/spaam/svtplay-dl) github repository and update
the value of `SVTPLAY_VERSION` when a new version is available.

## Configuration Summary

Based on this config, Renovate will:

- Separate major versions of dependencies into individual branches/MRs
- Do not separate patch and minor upgrades into separate MRs for the same
  dependency
- Upgrade to unstable versions only if the existing version is unstable
- Raise MRs immediately (after branch is created)
- If semantic commits detected, use semantic commit type fix for dependencies
  and chore for all others
- Keep existing branches updated even when not scheduled
- Disable automerging feature - wait for humans to merge all MRs
- Ignore node_modules, bower_components, vendor and various test/tests
  directories
- Autodetect whether to pin dependencies or maintain ranges
- Rate limit MR creation to a maximum of two per hour
- Limit to maximum 20 open MRs at any time
- Group known monorepo packages together
- Use curated list of recommended non-monorepo package groupings
- Ignore spring cloud 1.x releases
- Ignore http4s digest-based 1.x milestones
- Enable Renovate Dependency Dashboard creation
- Run Renovate on following schedule: after 9am and before 5pm every weekday
