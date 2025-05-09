+++
title = "Attribute commits from unclaimed emails to yourself"
date = "2025-05-09T17:38:00-05:00"
draft = true
+++

GitHub attributes commits to a GitHub account via the commit email.
I noticed that it doesn't check whether that email was verified, so we can easily take over someone's unclaimed email.

## Reproduction

I made a Git repository with a single commit, made by `john.doe123456@example.com` (`john.doe@example.com` was in use‽).

```
$ git show
commit b8490d5efdcddeb48456fb72b90d477c9fd00b96
Author: John Doe <john.doe123456@example.com>
Date:   Fri May 9 17:45:47 2025 -0400

    add line to README

diff --git a/README b/README
new file mode 100644
index 0000000..f769b61
--- /dev/null
+++ b/README
@@ -0,0 +1 @@
+Thank you for reading me.
```

Correctly, this commit is attributed to `John Doe`, and not a specific GitHub account:
![GitHub's repository page (logged out) showing the single commit attributed to John Doe](./no-attribution.png)

Now, I'll go add `john.doe123456@example.com` to my account:
![GitHub showing that my new email is not verified](./add-email.png)

Nevermind, it's not used lol
