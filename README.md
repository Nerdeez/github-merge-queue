
# Github Merge Queue Example

This repository is an example of github merge queue integration.
View the full lesson at academeez.com

## The problem

When multiple pull requests target the same branch (e.g., `main`), they can create conflicts even if they don't directly conflict with each other. Here's the scenario:

1. **Merge race conditions**: PR A and PR B both pass CI checks against the current state of `main`
2. PR A merges first, updating `main`
3. PR B's merge now fails because it was tested against an older version of `main` that didn't include PR A's changes
4. PR B needs to be rebased, CI needs to re-run, and the process repeats

This creates a frustrating cycle where:
- Developers must constantly rebase their PRs as new changes merge ahead of them
- CI pipelines run multiple times for the same PR
- The merge process is slow and inefficient
- There's no guarantee that a passing PR will actually merge successfully

**Merge queues solve this problem** by ensuring each PR is tested against the latest state of the target branch (including queued PRs ahead of it) before merging, preventing merge conflicts and reducing unnecessary CI runs.

## What is Merge Queue

A **Merge Queue** is a GitHub feature that manages the order and testing of pull requests before they merge into a protected branch. Instead of allowing PRs to merge immediately after approval and CI passes, merge queues create a sequential queue system.

Here's how it works:

1. **Enqueuing**: When a PR is approved and meets all branch protection requirements, it can be added to the merge queue by clicking "Merge when ready" or similar.

2. **Temporary branch creation**: GitHub creates a temporary branch that combines:
   - The current state of the target branch (e.g., `main`)
   - All PRs ahead in the queue (in order)
   - Your PR's changes

3. **CI validation**: GitHub runs your CI checks against this temporary branch, ensuring your PR works with all queued changes ahead of it.

4. **Sequential merging**: If CI passes, GitHub merges PRs one by one in queue order. If CI fails, your PR is removed from the queue and you're notified.

5. **Automatic retry**: If a PR ahead of yours in the queue fails or is removed, GitHub automatically re-runs your PR's checks against the updated queue state.

This ensures that every PR is validated against the exact state it will encounter when merged, eliminating merge conflicts and the need for constant rebasing.

## Simple example

In this simple example, we will present 2 workflows:

1. **`build.yaml`** - Runs with `pull_request` trigger
2. **`test.yaml`** - Runs after the build is complete

Since `test.yaml` will not run on `pull_request` trigger, it will generate a check using the GitHub API.

Each workflow follows the same pattern:
- Sleeps for a period of time
- Passes or fails depending on the content of result files:
  - `build.result.txt` - determines build workflow success/failure
  - `test.result.txt` - determines test workflow success/failure

This setup demonstrates how merge queues handle sequential workflows and how each PR is validated against the combined state of all queued PRs.