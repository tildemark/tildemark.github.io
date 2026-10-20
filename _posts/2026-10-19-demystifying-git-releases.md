---
layout: post
title: "Demystifying Git Releases: Tags, Branches, and Handling Production Hotfixes"
date: 2026-10-19 08:25:00 +0800
categories: [DevOps, Git]
tags: [git, gitea, github, workflow, devops, release-management]
image:
  path: https://cdn.sanchez.ph/blog/git-releases-preview.webp
  alt: "Demystifying Git Releases: Visual workflow of master branch, version tags v1.5.0 and v1.5.1, and production hotfixes"
---

If you are working on a large-scale project like an Enterprise Resource Planning (ERP) system, managing how code moves from a developer's laptop to a live production server is critical. 

Often, teams run into friction when trying to understand the difference between branches, tags, milestones, and releases. Developers might ask: *"Do I submit my Pull Request to `master` or to `v1.5`?"* or *"If we are working on `v1.6` but a bug appears in `v1.5`, how do we fix it without releasing unfinished code?"*

In this post, we will demystify how Git releases actually work, explore the ideal branching strategy, and walk through real-world scenarios like hotfixes and cherry-picking.

---

## 1. The Core Concepts: Branches vs. Tags

To understand release management, you first need to stop thinking of version numbers (`v1.5.0`) as branches. 

Think of your Git repository as a factory with a conveyor belt:

*   **A Commit:** A permanent save point of your code (like a single box on the belt).
*   **A Branch (`master`):** The moving conveyor belt itself. Every time a developer merges a Pull Request (PR), a new box is added. `master` is a live, never-ending stream. It represents the *latest* state of work.
*   **A Tag (`v1.5.0`):** A permanent, unmoving serial number stamped onto one specific box on the conveyor belt. 

![The Factory Conveyor Belt Analogy: Master Branch vs. Git Tags](https://cdn.sanchez.ph/blog/git-branches-vs-tags.webp)
_The `master` branch acts as a continuous conveyor belt of commits, while a Git Tag (`v1.5.0`) stamps a permanent snapshot for production._

**The Golden Rule:** You cannot submit a Pull Request to a Tag. A tag is a frozen photograph of history. You submit PRs to branches (like `master`), and DevOps creates tags from those branches when the code is ready to ship.

### Milestones vs. Git Tags
In platforms like Gitea or GitHub, you will see both "Milestones" and "Releases/Tags". They are entirely different things:

| Feature       | What is it?                                         | Does it change code?                                        | Who is it for?                             |
| :------------ | :-------------------------------------------------- | :---------------------------------------------------------- | :----------------------------------------- |
| **Milestone** | A digital to-do list or folder for tracking issues. | **No.** It is purely visual bookkeeping.                    | Project managers tracking sprint progress. |
| **Git Tag**   | A technical snapshot of code in Git.                | **Yes.** This is what production servers actually download. | CI/CD pipelines, DevOps, and servers.      |

If you assign 20 issues to a Milestone called `v1.5.0` but never create the Git tag, nothing happens on production. The **Tag** is the absolute basis for any release.

---

## 2. The Ideal Workflow: Trunk with Release Branches

For continuously developed software like an ERP, the **"Trunk with Release Branches"** pattern is the most stable and modern approach.

![Trunk with Release Branches Workflow](https://cdn.sanchez.ph/blog/git-trunk-release-workflow.webp)
_Feature branches merge into `master`, and DevOps stamps immutable release tags (`v1.5.0`) for production while `master` continues toward `v1.6.0`._

### The Day-to-Day (Building Features)
1.  **Branch off:** A developer starts a new feature by creating a branch from `master` (e.g., `feature/tax-calc`).
2.  **Commit and PR:** They write code, commit, and open a PR targeting `master`.
3.  **Merge:** Once approved, the code merges into `master`. 

### How a Release is Made
When the sprint ends and management decides the features on `master` constitute version `1.5.0`, DevOps steps in:

1.  DevOps looks at the exact state of `master`.
2.  They stamp a **Tag** named `v1.5.0` onto the latest commit.
3.  In Gitea/GitHub, they create a Release pointing to that tag.
4.  The production server is instructed to download and run tag `v1.5.0`.

At this moment, production is safely frozen on `1.5.0`, and developers can immediately go back to merging new PRs into `master` for version `1.6.0`.

---

## 3. Real-World Scenarios & Emergencies

What happens when things don't go according to plan? Here is how to handle the most common Git headaches.

![Production Hotfix vs. Cherry-Picking Workflow](https://cdn.sanchez.ph/blog/git-hotfix-and-cherry-pick.webp)
_Comparing emergency workflows: branching a hotfix directly off tag `v1.5.0` vs. cherry-picking a single commit (`a7c3f19`) into `release/v1.5.1`._

### Scenario A: The Production Hotfix
**The Problem:** Production is running `v1.5.0`. A critical bug is found. Meanwhile, developers have already merged 10 new, unfinished features into `master` for `v1.6.0`. You cannot deploy `master` to fix the bug, or you will deploy half-finished features.

**The Solution (Branching off a Tag):**
1.  **Jump back in time:** A developer creates a temporary branch starting directly from the frozen production tag:
    ```bash
    git checkout -b hotfix/fix-crash v1.5.0
    ```
2.  **Fix it:** The developer writes the fix for the bug *only*.
3.  **Tag and Deploy:** DevOps tests this tiny branch, stamps a new tag `v1.5.1`, and deploys it. Production is fixed!
4.  **Sync `master`:** The hotfix branch is merged back into `master` so the bug doesn't reappear when `1.6.0` launches.

### Scenario B: The "Need it Now" Feature (Cherry-Picking)
**The Problem:** The team is halfway through the `1.6.0` sprint. Developer Alice finishes a highly requested Export Report and merges it into `master`. Management says, *"We can't wait for 1.6.0. Put Alice's report into a 1.5.1 patch release today, but don't include the other 1.6.0 stuff!"*

**The Solution (Cherry-Picking):**
Git allows you to copy a specific commit from one branch and paste it into another.

1.  Find the commit hash of Alice's merged PR (e.g., `a7c3f19`).
2.  Create a release branch from production: `git checkout -b release/v1.5.1 v1.5.0`.
3.  **Cherry-pick** the commit:
    ```bash
    git cherry-pick a7c3f19
    ```
4.  Tag this branch as `v1.5.1` and deploy it. You have successfully extracted one specific feature from `master` and shipped it early.

### Scenario C: Pre-Planned Sprints and Incomplete Work
You plan a sprint with a Milestone for `v1.5.0` containing 15 issues. By release day, only 13 are merged into `master`.

**How to handle it:** 
Never merge incomplete code just to hit a milestone. Simply edit the 2 unfinished issues in Gitea and move them to the `v1.6.0` milestone. DevOps tags `master` as `v1.5.0` with the 13 completed features. The milestone is just bookkeeping—it adapts to reality.

---

## Summary: The Golden Rules for Your Team

To keep your ERP repository clean and your releases stress-free, enforce these rules:

1.  **Developers always target `master`:** Your everyday PRs go to the main conveyor belt.
2.  **Never target a Tag:** Tags are frozen. You cannot push to them.
3.  **Leave tagging to DevOps:** Developers merge code; DevOps decides when to stamp a tag and deploy.
4.  **Protect your branches:** Use your Git server settings to require PR reviews before anything enters `master`.

By treating `master` as a continuous stream of work and Tags as immutable historical snapshots, your team can build rapidly without ever jeopardizing the stability of your production environment.