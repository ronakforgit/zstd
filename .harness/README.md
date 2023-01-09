
## Build & Test facebook/zstd on Harness CI

This is a fork of the [facebook/zstd](https://github.com/facebook/zstd) project. This project is used for testing Harness CI's capabilities by the CME team at Harness. This file contains instructions on how to run [facebook/zstd](https://github.com/facebook/zstd) on Harness CI.

*   [Harness Fast CI Blog Announcement](https://harness.io/blog/announcing-speed-enhancements-and-hosted-builds-for-harness-ci)
*   [Get Started with Harness CI](https://harness.io/products/continuous-integration)

## Setting up this pipeline on Harness CI Hosted Builds

1.  Create a [GitHub Account](https://github.com) or use an existing one
2.  Fork [this repository](https://github.com/facebook/zstd/fork) into your GitHub account.
3.  If you are new to Harness CI, signup for [Harness CI](https://app.harness.io/auth/#/signup)
    *   Select the `Continuous Integration` module and choose the `Starter pipeline` wizard to create your first pipeline using the forked repo from #2.
    *   Go to the newly created pipeline and hit the `Triggers`tab. If everything went well, you should see two triggers auto-created. A `Pull Request`trigger and a `Push`trigger. For this exercise, we only need `Pull Request`trigger to be enabled. So, please disable or delete the `Push`trigger.
4.  If you are an existing Harness CI user, create a new pipeline to use the cloud option for infrastructure and set up the PR trigger.
5.  Add the pipeline.yaml stages in the YAML editor:

```plaintext
  stages:
    - parallel:
        - stage:
            name: short-tests-0
            identifier: shorttests0
            type: CI
            spec:
              cloneCodebase: true
              platform:
                os: Linux
                arch: Amd64
              runtime:
                type: Cloud
                spec: {}
              execution:
                steps:
                  - step:
                      type: Run
                      name: test
                      identifier: test
                      spec:
                        connectorRef: account.harnessImage
                        image: fbopensource/zstd-circleci-primary:0.0.1
                        shell: Bash
                        command: |-
                          ./tests/test-license.py
                                      cc -v
                                      CFLAGS="-O0 -Werror -pedantic" 
                                      sudo make allmost; sudo make clean
                                      sudo make c99build; sudo make clean
                                      sudo make c11build; sudo make clean
                                      sudo make -j regressiontest; sudo make clean
                                      sudo make shortest; sudo make clean
                                      sudo make cxxtest; sudo make clean
        - stage:
            name: short-tests-1
            identifier: shorttests1
            type: CI
            spec:
              cloneCodebase: true
              platform:
                os: Linux
                arch: Amd64
              runtime:
                type: Cloud
                spec: {}
              execution:
                steps:
                  - step:
                      type: Run
                      name: test
                      identifier: test
                      spec:
                        connectorRef: account.harnessImage
                        image: fbopensource/zstd-circleci-primary:0.0.1
                        shell: Sh
                        command: |2-
                                      sudo make gnu90build; sudo make clean
                                      sudo make gnu99build; sudo make clean
                                      sudo make ppc64build V=1; sudo make clean
                                      sudo make ppcbuild   V=1; sudo make clean
                                      sudo make armbuild   V=1; sudo make clean
                                      sudo make aarch64build V=1; sudo make clean
                                      sudo make -C tests test-legacy test-longmatch; sudo make clean
                                      sudo make -C lib libzstd-nomt; sudo make clean
        - stage:
            name: regression-test
            identifier: regressiontest
            type: CI
            spec:
              cloneCodebase: true
              platform:
                os: Linux
                arch: Amd64
              runtime:
                type: Cloud
                spec: {}
              execution:
                steps:
                  - step:
                      type: Run
                      name: Regression Test
                      identifier: Regression_Test
                      spec:
                        connectorRef: account.harnessImage
                        image: fbopensource/zstd-circleci-primary:0.0.1
                        shell: Sh
                        command: |2-
                                      sudo make -C programs zstd
                                      sudo make -C tests/regression test
                                      sudo mkdir -p $CIRCLE_ARTIFACTS
                                      sudo ./tests/regression/test                     \
                                          --cache  tests/regression/cache         \
                                          --output $CIRCLE_ARTIFACTS/results.csv  \
                                          --zstd   programs/zstd
                                      echo "NOTE: The new results.csv is uploaded as an artifact to this job"
                                      echo "      If this fails, go to the Artifacts pane in CircleCI, "
                                      echo "      download /tmp/circleci-artifacts/results.csv, and if they "
                                      echo "      are still good, copy it into the repo and commit it."
                                      echo "> diff tests/regression/results.csv $CIRCLE_ARTIFACTS/results.csv"
                                      diff tests/regression/results.csv $CIRCLE_ARTIFACTS/results.csv
                        envVariables:
                          CIRCLE_ARTIFACTS: /tmp/circleci-artifacts

```

> Make sure you modify the connectors with the connectors you create.

6.  Create a `Pull Request` in a new branch by updating the project. (e.g. add a comment or new line). This should invoke a build on  Harness CI
7.  Merge the PR after the pipeline execution is successful.
8.  Enable Circle CI : The repository forked in Step 2 already has a Circle CI `config.yml`  file added. You can use it to set up a Circle CI pipeline .
