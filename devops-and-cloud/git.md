---
title: Git
parent: DevOps and Cloud
nav_order: 1
---

# Git
{: .no_toc }

Everyday Git questions.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Questions on this page</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## What is the difference between git merge and git rebase?

Easy
{: .label .label-green }

**Answer**

`merge` combines branches and keeps history with a merge commit. `rebase` replays your commits on top of another branch for a linear history but rewrites commits, so never rebase shared branches. Use `git pull --rebase` or squash merges to keep history clean.

---

<!--
ADD NEW QUESTIONS ABOVE THIS COMMENT. Copy the block below, then replace the text.
Levels: Easy (label-green), Medium (label-yellow), Hard (label-red).

## Your question here?

Medium
{: .label .label-yellow }

**Answer**

Write the answer here. Use tables, lists and ```java code blocks.

**Key points**
- Point one

**Follow-up questions**
- Follow-up one

Separate questions with a horizontal rule (three dashes on their own line).
-->
