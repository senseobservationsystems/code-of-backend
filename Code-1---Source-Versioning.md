# Code 1 - Source Versioning

## 1. Branching Strategy
   1. `master` branch shall contains only stable codebase.
   2. all new feature and bugfix shall be implemented in separate branch, branching out from `develop`
   3. hotfix shall be implemented in separate branch, branching out from `master`
   4. each line of release shall be implemented in separate branch, branching out from `develop`

## 2. Branch name
   1. the name of the branch shall comply with following rule
      1. the branch name shall start with implementation type, followed with short description of the implementation, and if exists, end with issue number.

         `<type>/<short description>#<issue_number>`
      2. implementation type shall be one of `feature`, `bugfix`, or `hotfix`
      3. short description part shall reflect what the change is mainly about in less than 3 words, separate with `_` (underscore)

         `feature/my_new_feature#123`
## 3. Commit message
   WIP