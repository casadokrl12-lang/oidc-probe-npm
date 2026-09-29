# oidc-probe-npm

Disposable public repository used to mint **real** GitHub Actions OIDC id_tokens
(`permissions: id-token: write`) in order to observe the exact claim set GitHub
emits, and to test the npm Trusted Publishing exchange endpoint
(`POST /-/npm/v1/oidc/token/exchange/package/<pkg>`) read-only.

No packages are published. No third-party state is modified.
