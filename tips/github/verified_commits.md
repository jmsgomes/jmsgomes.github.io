# Verified GitHub Commits

If your commits were not configured to be signed, then they will as `Unverified`
when **Vigilant Mode** is turned on.

> [!CAUTION]
>
> If this occurs on a branch such as `main` where others collaborate,
> then collaborators will be forced to re-clone the repo!

1. Check out the branch with the unsigned commits.

2. Review the commits that are unsigned (or not validly signed), then **copy the
   id of the _latest commit prior to the unsigned_**.

   ```shell
   git log --show-signature
   ```

   ```shell
   LAST_GOOD_COMMIT=
   ```

   > [!TIP]
   >
   > If there was no good commit, set `LAST_GOOD_COMMIT='--root'`

3. Run the following rebase command:

   ```shell
   git rebase --exec 'git commit --amend --no-edit -n -S' -i "$LAST_GOOD_COMMIT"
   ```

4. Inspect the commits to confirm they are signed:

   ```shell
   git log --show-signature
   ```

5. Forcefully push the amended commits:

   ```shell
   git push --force-with-lease
   ```

   > [!WARNING]
   >
   > Skip if prompted to pull or fetch.
