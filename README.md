# renovate-config

I want to manage multiple repositories using Renovate and want the same custom
config across all or most of them, hence I have a preset config so that we can
"extend" it in every applicable repository. This way when I want to change the
Renovate configuration I can make the change in one location rather than having
to copy/paste it to every repository individually.

The [default.json](./default.json) file in this repository is the preset config
I use in all repositories.

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

## Ignore certain versions

Sometimes it is necessary to ignore certain versions. One good example is
`linuxserver/docker-qbittorrent` which changed the tagging convention from
`14.3.9.99202110311443-7435-01519b5e7ubuntu20.04.1-ls166` to `4.4.5-r0-ls213`.

We can ignore the old format by using:

```json
{
  "matchPackageNames": ["linuxserver/docker-qbittorrent"],
  "allowedVersions": "!/\\d+.[2-3].\\d+.\\d{14}-\\d{4}-.*ubuntu.*-ls\\d+$/"
}
```
