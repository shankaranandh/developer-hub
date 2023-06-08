---
sidebar_position: 6
title: Swift application
description: Use a CI pipeline to build and test a Swift application.
keywords: [Hosted Build, Continuous Integration, Hosted, CI Tutorial]
slug: /ci-pipelines/build/swift
---

```mdx-code-block
import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
import CISignupTip from '/tutorials/shared/ci-signup-tip.md';
```

<ctabanner
  buttonText="Learn More"
  title="Continue your learning journey."
  tagline="Take a Continuous Integration Certification today!"
  link="/certifications/continuous-integration"
  closable={true}
  target="_self"
/>

You can build and test a [Swift](https://developer.apple.com/documentation/xcode/building-swift-packages-or-apps-that-use-them-in-continuous-integration-workflows/) application using a macOS platform on [Harness Cloud](/docs/continuous-integration/use-ci/set-up-build-infrastructure/use-harness-cloud-build-infrastructure) or a [self-hosted VM](/docs/continuous-integration/use-ci/set-up-build-infrastructure/vm-build-infrastructure/define-macos-build-infra-with-anka-registry/) build infrastructure.

This guide assumes you've created a Harness CI pipeline. For more information about creating pipelines, go to:

* [CI pipeline creation overview](/docs/continuous-integration/use-ci/prep-ci-pipeline-components)
* [Harness Cloud pipeline tutorial](/tutorials/ci-pipelines/fastest-ci)
* [Kubernetes cluster pipeline tutorial](/tutorials/ci-pipelines/build/kubernetes-build-farm)

<CISignupTip />

To learn more about Apple M1 support for Harness CI pipelines, go to the [Apple M1 and Harness blog post](https://www.harness.io/blog/ios-build-pipelines-apple-m1).

## Cache dependencies

To cache your Swift dependencies, you can:

* [Save and Restore Cache from S3](/docs/continuous-integration/use-ci/caching-ci-data/saving-cache/)
* [Save and Restore Cache from GCS](/docs/continuous-integration/use-ci/caching-ci-data/save-cache-in-gcs)

Here's an example of a pipeline with **Save Cache to S3** and **Restore Cache from S3** steps.

```yaml
            steps:
              - step:
                  type: RestoreCacheS3
                  name: Restore Cache From S3
                  identifier: Restore_Cache_From_S3
                  spec:
                    connectorRef: AWS_Connector
                    region: us-east-1
                    bucket: your-s3-bucket
                    key: cache-{{ checksum filepath1 }}
                    archiveFormat: Tar
              - step:
                  type: Run
                  ...
              - step:
                  type: BuildAndPushDockerRegistry
                  ...
              - step:
                  type: SaveCacheS3
                  name: Save Cache to S3
                  identifier: Save_Cache_to_S3
                  spec:
                    connectorRef: AWS_Connector
                    region: us-east-1
                    bucket: your-s3-bucket
                    key: cache-{{ checksum filepath1 }}
                    sourcePaths:
                      - directory1
                      - directory2
                    archiveFormat: Tar
```

## Build and run tests

Add [Run steps](/docs/continuous-integration/use-ci/run-ci-scripts/run-step-settings/) to build and [run tests in Harness CI](/docs/continuous-integration/use-ci/set-up-test-intelligence/run-tests-in-ci).

```mdx-code-block
<Tabs>
<TabItem value="Harness Cloud">
```

```yaml
              - step:
                  type: Run
                  name: test with swift
                  identifier: test_with_swift
                  spec:
                    shell: Sh
                    command: |-
                      swift build
                      swift test
```

```mdx-code-block
</TabItem>

<TabItem value="Self-hosted">
```

```yaml
              - step:
                  type: Run
                  name: Run xcode
                  identifier: Run_xcode
                  spec:
                    connectorRef: account.harnessImage
                    image: swift:latest
                    shell: Sh
                    command: |-
                      swift build
                      swift test
```

```mdx-code-block
</TabItem>
</Tabs>
```

### Visualize test results

If you want to [view test results in Harness](/docs/continuous-integration/use-ci/set-up-test-intelligence/viewing-tests/), your test reports must be in JUnit XML format, and your steps that run tests must include the `reports` specification. Use a tool such as the [SwiftTestReporter](https://swiftpackageindex.com/allegro/swift-junit) to produce correctly-formatted reports.

```yaml
                    reports:
                      type: JUnit
                      spec:
                        paths:
                          - report.xml
```

## Specify version

The Apple App Store requires that you use a version of Swift that is included with Xcode. Using another version of Swift can make your app ineligible for the App Store.

```mdx-code-block
<Tabs>
  <TabItem value="hosted" label="Harness Cloud" default>
```

You can use commands in a [Run step](/docs/continuous-integration/use-ci/run-ci-scripts/run-step-settings) to [download](https://www.swift.org/download/) and [install](https://www.swift.org/getting-started/) Swift.

```mdx-code-block
  </TabItem>
  <TabItem value="selfhosted" label="Self-hosted">
```

Specify the desired [Swift Docker image](https://hub.docker.com/_/swift) tag in your steps. There is no need for a separate install step when using Docker.

<details>
<summary>Use a specific Swift version</summary>

```yaml
              - step:
                  type: Run
                  name: Swift Version
                  identifier: swiftversion
                  spec:
                    connectorRef: account.harnessImage
                    image: swift:5.3.3
                    shell: Sh
                    command: |-
                      swift --version
```

</details>

<details>
<summary>Use multiple Swift versions</summary>

1. Add the [matrix looping strategy](/docs/platform/pipelines/looping-strategies-matrix-repeat-and-parallelism/) configuration to your stage.

```yaml
    - stage:
        strategy:
          matrix:
            swiftVersion:
              - 18.16.0
              - 20.2.0
```

2. Reference the matrix variable in the `image` field of your steps.

```yaml
              - step:
                  type: Run
                  name: Swift Version
                  identifier: swiftversion
                  spec:
                    connectorRef: account.harnessImage
                    image: node:<+matrix.swiftVersion>
                    shell: Sh
                    command: |-
                      swift --version
```

</details>

```mdx-code-block
  </TabItem>
</Tabs>
```

## Next steps

Now that you have created a pipeline that builds and tests a Swift app, you could:

* Create [triggers](/docs/category/triggers) to automatically run your pipeline.
* Add steps to [build and upload artifacts](/docs/category/build-and-upload-artifacts).
* Add a step to [build and push an image to a Docker registry](/docs/continuous-integration/use-ci/build-and-upload-artifacts/build-and-push-to-docker-hub-step-settings/).
* Explore other ways to [optimize and enhance CI pipelines](/docs/continuous-integration/use-ci/optimize-and-more/optimizing-ci-build-times).