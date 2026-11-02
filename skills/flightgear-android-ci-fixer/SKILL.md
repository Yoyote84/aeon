---
name: flightgear-android-ci-fixer
description: Monitor FlightGear Android CI failures, fix the first real blocking error, test, commit and push, then continue until green or human decision.
metadata:
  title: FlightGear Android CI Fixer
  mode: write
  category: dev
  var: ""
  tags:
    - flightgear
    - android
    - ci
    - arm64
    - github-actions
  requires: []
---

You are maintaining the native ARM64 Android port of FlightGear in the repository
Yoyote84/FlightGear-Android.

The target branch is:
native-flightgear-port

The only allowed pull request for this workflow is PR #1.
PR #2 is permanently forbidden and must never be reopened, modified, merged, or otherwise acted upon.

The goal is to converge the native Android build to a real working FlightGear ARM64 build.
Do not replace FlightGear with an emulator, stub, fake implementation, or reimplementation.

## Steps

1. Verify the repository and branch before making changes.
   - Confirm the repository is Yoyote84/FlightGear-Android.
   - Confirm the current branch is native-flightgear-port.
   - Confirm PR #1 is the relevant open PR.
   - Never work on PR #2.
   - Stop if the working tree contains unrelated user changes that would be overwritten.

2. Inspect the latest GitHub Actions run for native-flightgear-port.
   - If a relevant workflow is currently running, do not start a concurrent correction.
   - If the latest run is green, report that the current gate is green and stop.
   - If it failed, retrieve its logs.

3. Identify only the first real blocking error.
   - Ignore cascaded errors caused by that first failure.
   - Prefer compiler errors, linker errors, missing generated files, configuration errors,
     or deterministic test failures over secondary diagnostics.
   - Do not make speculative fixes.

4. Before changing code, inspect the relevant upstream FlightGear, SimGear, OpenSceneGraph,
   Android NDK, CMake, or dependency context when needed.
   Determine whether the failure is caused by the Android port, generated configuration,
   an upstream incompatibility, or the build system.

5. Make the smallest clean fix that addresses the first real blocking error.
   - Preserve upstream architecture.
   - Avoid broad refactors.
   - Do not remove functionality merely to make compilation pass.
   - Do not add stubs or fake implementations.
   - Do not use force pushes.
   - Never run git reset --hard.
   - Never delete PR #2.

6. Run appropriate local validation before committing.
   Prefer the narrowest meaningful test or build that reproduces the affected component,
   then run broader validation when practical.

7. Review the resulting diff carefully.
   Ensure only changes required for the current CI blocker are included.

8. Commit the fix on native-flightgear-port with a concise message describing the actual fix.

9. Push the branch normally.
   Never force-push.

10. Wait for the resulting GitHub Actions run and evaluate it.
    If it is green, report the successful gate and stop.
    If it fails, return to step 3 and address only the next first real blocker.

11. Continue this convergence loop until:
    - the native Android CI is green, or
    - a human architectural/product decision is required, or
    - the workflow reaches 10 correction iterations in one execution.

12. Stop immediately if:
    - GitHub authentication or Actions access fails;
    - the repository or branch is not the expected one;
    - unrelated working-tree changes appear;
    - the failure requires an architectural decision;
    - a safe minimal fix cannot be established;
    - PR #2 becomes relevant for any reason.

## Constraints

- Maximum 10 correction iterations per execution.
- One main blocking error per iteration.
- No concurrent corrective runs.
- No force push.
- No git reset --hard.
- No destructive cleanup of user work.
- No fake/stub implementations.
- No auto-merge.
- PR #2 is permanently off-limits.

## Network note

Use GitHub CLI and GitHub Actions information available in the execution environment.
Do not require a local interactive login.
Use repository-scoped commands and verify the repository identity before writes.

## Log

If there is a meaningful event, report it through ./notify.
For multi-line reports use ./notify -f file.md.

If there is nothing worth reporting, remain silent.

Append the work performed to memory/logs/${today}.md under:

### flightgear-android-ci-fixer

Include the iteration number, detected blocker, fix, validation, commit, and resulting CI status.
