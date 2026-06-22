Update: Qwen3.6 27b and 35b-a3b -> king





To fix this error, you need to ==**replace your GitHub account password with a Personal Access Token (PAT)** or switch to SSH authentication==. GitHub completely removed support for account passwords during Git operations to improve security. [[1](https://medium.com/@yoshiyuki.watanabe/why-git-clone-fails-with-password-authentication-is-not-supported-on-aws-cloudshell-68b1a779a17e), [2](https://dev.to/nutan_mishra_95d0a3808a4d/fixing-github-authentication-error-erase-operation-not-supported-and-invalid-username-or-token-47jp), [3](https://www.cnblogs.com/zhangzhihui/p/19070736), [4](https://dev.to/kkibet/solving-git-authentication-failures-password-authentication-is-not-supported-error-n96)]

Follow these steps to generate a token and update your computer's cached credentials. [[1](https://medium.com/@rahulsharan512/how-to-connect-git-bash-to-your-github-account-two-easy-methods-a332e037dea5), [2](https://stackoverflow.com/questions/45903106/github-enterprise-remote-password-authentication-is-not-available-for-git-ope)]

Step 1: Generate a Personal Access Token (PAT) [[1](https://coda.io/@peter-sigurdson/to-resolve-the-authentication-issue-youre-experiencing-with-push)]

1. Log into your account on [GitHub](https://github.com/).
2. In the top-right corner, click your profile photo and select **Settings**.
3. Scroll down the left sidebar and click **Developer settings**.
4. Expand **Personal access tokens** and select **Tokens (classic)**.
5. Click **Generate new token** → **Generate new token (classic)**.
6. Give your token a descriptive name (e.g., "Laptop Git Access") and choose an expiration date.
7. Select the **`repo`** checkbox (this grants control over your repositories).
8. Click **Generate token** at the bottom of the page.
9. **Copy the token immediately.** You cannot see it again once you leave the page. [[1](https://www.reddit.com/r/learnprogramming/comments/1mq2ypp/password_authentication_is_not_supported_for_git/), [2](https://stackoverflow.com/questions/45903106/github-enterprise-remote-password-authentication-is-not-available-for-git-ope), [3](https://www.youtube.com/watch?v=BiPuDyZy74g), [4](https://www.youtube.com/watch?v=ePCBuIQJAUc), [5](https://dev.to/nutan_mishra_95d0a3808a4d/fixing-github-authentication-error-erase-operation-not-supported-and-invalid-username-or-token-47jp), [6](https://medium.com/@yoshiyuki.watanabe/why-git-clone-fails-with-password-authentication-is-not-supported-on-aws-cloudshell-68b1a779a17e), [7](https://dev.to/kkibet/solving-git-authentication-failures-password-authentication-is-not-supported-error-n96)]

Step 2: Clear and Update Cached Credentials [[1](https://cloudxlab.com/assessment/displayslide/4951/github-project-getting-started-with-git-push-your-repository-to-remote-server)]

Your operating system has likely cached your old password. You need to clear it so Git prompts you for the new token. [[1](https://stackoverflow.com/questions/45903106/github-enterprise-remote-password-authentication-is-not-available-for-git-ope), [2](https://dev.to/nutan_mishra_95d0a3808a4d/fixing-github-authentication-error-erase-operation-not-supported-and-invalid-username-or-token-47jp), [3](https://www.geeksforgeeks.org/git/how-to-fix-git-authentication-failed-error/)]

For Windows Users:

1. Open the Windows Start Menu, search for **Credential Manager**, and open it.
2. Select **Windows Credentials**.
3. Scroll down to the **Generic Credentials** section and locate entries starting with `git:https://github.com`.
4. Click **Remove** on those entries.
5. The next time you perform a `git push` or `git pull`, a login box will appear. Use your standard GitHub username, but **paste the PAT into the password field**. [[1](https://stackoverflow.com/questions/45903106/github-enterprise-remote-password-authentication-is-not-available-for-git-ope), [2](https://stackoverflow.com/questions/71600196/github-remote-invalid-username-or-password-fatal-authentication-failed), [3](https://dev.to/nutan_mishra_95d0a3808a4d/fixing-github-authentication-error-erase-operation-not-supported-and-invalid-username-or-token-47jp), [4](https://github.com/orgs/community/discussions/171965), [5](https://medium.com/@yoshiyuki.watanabe/why-git-clone-fails-with-password-authentication-is-not-supported-on-aws-cloudshell-68b1a779a17e)]

For macOS Users:

1. Open **Keychain Access** (via Spotlight Search with `Cmd + Space`).
2. Search for `github.com` in the top right search bar.
3. Find the **internet password** entry for your GitHub account, right-click it, and choose **Delete**.
4. Return to your terminal and execute your Git command. Provide your GitHub username, and **paste the PAT when prompted for your password**. [[1](https://www.reddit.com/r/learnprogramming/comments/1mq2ypp/password_authentication_is_not_supported_for_git/), [2](https://dev.to/nutan_mishra_95d0a3808a4d/fixing-github-authentication-error-erase-operation-not-supported-and-invalid-username-or-token-47jp), [3](https://medium.com/@yoshiyuki.watanabe/why-git-clone-fails-with-password-authentication-is-not-supported-on-aws-cloudshell-68b1a779a17e), [4](https://github.com/orgs/community/discussions/171965), [5](https://stackoverflow.com/questions/21615431/git-pushes-with-wrong-user-from-terminal)]

Quick Terminal Alternative (All Platforms):

If you do not want to use your operating system's credential UI, force Git to use the token directly by embedding it into your project's remote URL: [[1](https://www.youtube.com/watch?v=BiPuDyZy74g), [2](https://www.youtube.com/watch?v=NRSX7RcmPEY&t=162), [3](https://www.geeksforgeeks.org/git/how-to-fix-git-authentication-failed-error/)]

bash

```
git remote set-url origin https://<YOUR_GITHUB_USERNAME>:<YOUR_NEW_TOKEN>@github.com/<YOUR_ORGANIZATION_OR_USERNAME>/<YOUR_REPO_NAME>.git
```

Use code with caution.

_Replace the placeholders with your actual username, the generated token, and your repository path._ [[1](https://www.youtube.com/watch?v=ePCBuIQJAUc), [2](https://www.youtube.com/watch?v=BiPuDyZy74g)]

Would you like help switching to **SSH authentication** instead, so you never have to deal with tokens or passwords in your terminal again? [[1](https://dev.to/kkibet/solving-git-authentication-failures-password-authentication-is-not-supported-error-n96), [2](https://github.com/orgs/community/discussions/169227)]