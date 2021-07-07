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
annotate them using comments, which the regex manager will end up parsing:

```
# renovate: datasource=github-releases depName=hashicorp/terraform versioning=loose
ENV TERRAFORM_VERSION="1.0.0"
```

The example above will look for the newest version of `terraform` within the
`hashicorp/terraform` github repository.

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
