### What's changed in v1.3.0

* feat: e2e failure debugging steps + cleanup (#20) (by @patrickleet)

  When an e2e test suite fails, before exiting, we log k8s events, managed resources, get -o yaml / describe for unhealthy managed resources, and get-o yaml / describe for user inputted types / detected type from provided example.

  Also, when e2e test fails, cleanup previously did not get initialized, and now there is a step to force the cleanup process to begin.


See full diff: [v1.2.0...v1.3.0](https://github.com/unbounded-tech/workflows-crossplane/compare/v1.2.0...v1.3.0)
