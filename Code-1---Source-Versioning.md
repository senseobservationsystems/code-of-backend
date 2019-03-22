# Code 1 - Source Versioning

## 1. Branching Strategy
   1. `master` branch shall contains stable codebase only.
   2. all new feature and bugfix shall be implemented in separate branch, branching out from `develop`
   3. hotfix shall be implemented in separate branch, branching out from `master`
   4. each line of release shall be implemented in separate branch, branching out from `develop`

## 2. Branch name
   1. Branch can be in one of following implementation type:
      - `feature`, when the implementation add, change, or remove a feature from the overall project
      - `bugfix`, when the implementation fix a bug in the development branch `develop`
      - `hotfix`, when the implementation fix a bug in production branch `master`
      - `refactor`, when the implementation does not change any use case (feature nor bug)
      - `release`, is a special branch that is exclusively used for release process

   2. All branch shall have `short description` in the name that reflect what the change is mainly about, short enough (3 words or less), separate with `_` (underscore)

          my_new_feature

      1. Specific for `release` implementation type, the `short description` shall formatted as follow

             <major version>.<minor version>.<patch number>

   2. When the branch is stand alone implementation (not connected to any Issue), the branch name must confirm with following pattern

          <type>/<short description>

      example: `feature/network_deletion_time`

   3. When the branch is connected to an github Issue, the branch name must conform with following pattern

          <type>/<short description>#<issue_number>

      example: `feature/network_deletion_time#123` where 123 the issue number that the branch connected to

## 3. Commit message
   WIP