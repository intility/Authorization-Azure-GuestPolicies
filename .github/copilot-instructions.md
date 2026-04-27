# Repository instructions for GitHub Copilot

## Project

Multi-target NuGet library: `Intility.Authorization.Azure.GuestPolicies`. Targets `net8.0`, `net9.0`, and `net10.0`. Each TFM pins ASP.NET Core packages to its own major version.

## Critical review rule for dependency updates

**Do not allow `Microsoft.AspNetCore.Authorization` (or any package referenced under a TFM-conditional `<ItemGroup>`) to bump across major versions for a TFM that does not match.**

The `csproj` uses `<ItemGroup Condition=" '$(TargetFramework)' == 'netX.0' ">` blocks. Each block must reference the package version whose major matches that TFM:

| TFM | Allowed major version |
|-----|----------------------|
| `net8.0` | `8.x` only |
| `net9.0` | `9.x` only |
| `net10.0` | `10.x` only |

**This is a hard backward-compatibility constraint.** A `net8.0` consumer must not be forced to load 9.x or 10.x of `Microsoft.AspNetCore.Authorization`.

Dependabot cannot reason about TFM-conditional `PackageReference` items and will sometimes propose a single version for every TFM. Past incident: PR #49 bumped the `net8.0` line from `8.0.24` to `9.0.15`, breaking compat. It was reverted. **Block any PR that repeats this pattern.**

## Review checklist for dependency PRs

When reviewing a Dependabot or manual dependency PR, flag the PR as needing changes if any of these are true:

1. A TFM-conditional `PackageReference` has its major version changed to one that does not match the TFM.
2. All TFM-conditional `<ItemGroup>` blocks for the same package end up with the same version (this is almost always wrong for this repo).
3. A new `PackageReference` is added outside a TFM-conditional `<ItemGroup>` for a package that already has TFM-conditional entries.
4. The build matrix in `.github/workflows/build_and_test.yaml` is changed without a corresponding change to `<TargetFrameworks>` in the csproj (or vice versa).

## Other conventions

- Conventional Commits are required (release-please consumes them). `fix:` triggers patch, `feat:` triggers minor. Breaking changes use `!:` or `BREAKING CHANGE:` footer.
- GitHub Actions must be pinned to a full commit SHA, not a tag. Flag any `uses: owner/action@vX` (mutable ref) in workflow changes.
- Keep `permissions:` blocks minimal in workflows.
