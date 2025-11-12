#tips #reference #github #work/groww 


- Create ssh keys for each account. You can go through these to create ssh keys.
 https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent

<br>

- Add ssh keys to your github account.  https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account

<br>

- Edit your ~/.ssh/config file correctly.

```
Host github-groww-enterprise
    HostName github.com
    IdentityFile ~/.ssh/GrowwEnterpriseKey
    PreferredAuthentications publickey

Host github-sunny
  Hostname github.com
  AddKeysToAgent yes
  UseKeychain yes
  IdentityFile ~/.ssh/PersonalGithubKey
```

<br>

* Use git@Hostname while cloning or add remote url's etc. For example:
```
git@github-groww-enterprise:org/project.git => work github
git@github-sunny:org/project.git     => personal github
```

<br>

- Reference commands : https://chatgpt.com/share/67ab539d-c618-800c-a9f0-347d8fedcaa9

### References
1. https://gist.github.com/oanhnn/80a89405ab9023894df7
2. https://gist.github.com/rahularity/86da20fe3858e6b311de068201d279e3 