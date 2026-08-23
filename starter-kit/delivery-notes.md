# DevOps Delivery Notes

## Flow

Work should move through the development process in small, manageable changes rather than large batches. We can maintain good flow by using clear branch names such as `feature/starter-kit-files` and keeping pull requests focused on a single task. Smaller PRs make reviews faster and reduce the risk of merge conflicts or blocked work. This allows changes to move smoothly from development to review and eventually into the codebase.

## Feedback

Problems should be identified as early as possible rather than after changes reach production. Code reviews, automated tests, linting, and CI checks can provide quick feedback before a pull request is merged. Clear PR descriptions also help reviewers understand what changed and what should be checked. Early feedback reduces the cost of fixing mistakes and improves the overall quality of the system.

## Learning

At the end of each sprint, the team should review what worked well, what caused delays, and what could be improved. For example, if large pull requests caused slow reviews this sprint, we could break future work into smaller changes. We should also use failed CI checks, bugs, and review feedback to improve our development process. Applying these lessons in the next sprint creates a continuous cycle of improvement rather than repeating the same problems.

Idea → Branch → Code → Review → Merge → Delivery