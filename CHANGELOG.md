### What's changed in v2.17.0

* feat: delete extra resources (#35) (by @patrickleet)

  Added the ability to delete "extra resources" after test cleanup. For example, if you create an EKS cluster in extraResources in your e2e test, and want it to be cleaned up after. This gives you more control around deletion order.

  * `delete-extra-resources`: a JSON list of extra resource types to delete
  * `delete-extra-resources-timeout-minutes`: number of minutes to wait before timeout.

  ```
    e2e:
      uses: unbounded-tech/workflows-crossplane/.github/workflows/e2e.yaml@feat/delete-extra-resources
      with:
        aws: true
        aws-use-oidc: true
        aws-account-id: "034489662075"
        aws-region: us-east-2
        aws-role-duration-seconds: 7200
        debug-resource-types: |
          [
            "observes.stacks.hops.ops.com.ai"
          ]
        delete-extra-resources: |
          [
            "autoeksclusters.aws.hops.ops.com.ai"
          ]
        timeout-minutes: 44
        cleanup-timeout-minutes: 20
        delete-extra-resources-timeout-minutes: 20
  ```


See full diff: [v2.16.0...v2.17.0](https://github.com/unbounded-tech/workflows-crossplane/compare/v2.16.0...v2.17.0)
