### Configure git hook

1. copy file `pre-push` to `.git/hooks/` folder
2. make it executable `chmod +x .git/hooks/pre-push`
3. now this file will validate branch names while pushing to remote and it will only allow branches names starting with main-patch-, dev-patch-, prod-patch-, non-prod-patch-

In case, the branch naming convention fails you can rename the branch name by following this blog https://www.freecodecamp.org/news/git-rename-branch-how-to-change-a-local-branch-name/
