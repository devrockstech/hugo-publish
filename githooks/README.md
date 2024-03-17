### Configure git hook

1. copy file `pre-push` to `.git/hooks/` folder
2. make it executable `chmod +x .git/hooks/pre-push`
3. now this file will validate branch names while pushing to remote and it will only allow main-patch-*, dev-patch-*, prod-patch-*, non-prod-patch-*
