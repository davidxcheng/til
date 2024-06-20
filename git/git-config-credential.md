# Git credentials & Azure DevOps repos

The way git requests uses auth is still like magic to me on a good day, and more like a black box on bad days. Yesterday the magic stopped working and I had to spend a few hours to get it working again.

The issue began when I, via the git CLI, tried to push to the Azure DevOps git repo I was currently working on. The weird thing was that I had cloned 4 repos belonging to the same organisation and 2 of those gave me auth-issues by promting me for a password I didn't know. I had a vague memory och creating a Personal Access Token (PAT) at some point and when I checked Azure I hade several that were all still valid. And that added to my confusion because I was under the impression that git somehow submittet that PAT and didn't auth using username and password.

## Debugging

I tried logging in with `az devops login` which didn't help. Then I checked the remotes of the repos, using `git remote -v`, and realised that they used different host names even though they were all hosted in Azure DevOps (`dev.azure.com` vs `{organisation}.visualstudio.com`).

## KeyChain

After reading a lot of posts online I finally understood that the PAT (or whatever git uses for auth) was stored in a KeyChain (I'm on macOS). It took me a while to understand this because a while back, while I was on Windows, I had tracked down the PAT used for accessing a nuget feed to a file called `SessionTokenCache.dat` in the `%UserProfile%/AppData/Local/MicrosoftCredentialProvider` directory. So I assumed that it would be stored in a file on macOS too. I checked the KeyChain and it contained an entry for the host of the remotes that I could still access.

Now it was pretty clear that git somehow uses the entry in KeyChain to auth against DevOps. The problem was that the posts I was reading indicated that the way to get that magical entry in KeyChain was to pretty much do what I was doing, i.e. trying to push my code, and something called `git-credential-manager` would then display a UI where I signed into DevOps and stashed a PAT in the KeyChain so the magic could work another couple of months.

## git-credential

When git promted me for a password it was for a particular user which was the name of the DevOps organisation and the reason for that was that the username was included in the remote: `https://someorg@dev.azure.com/some/someorg/foo/_git/repo-name`. When I removed `someorg@` git promted me for both username and password which made it clear to me that git was using the default auth method.

So how do I get `git-credential-manager` (GCM) to take over and popup the DevOps login? I didn't have it installed (checked with `which git-credential-manager`) and therefore I first ignored it and thought I wouldn't need it since I'd somehow had things working until now. When reading about it I saw a lot of talk about setting `git.credential.helper` and other `git.credential` config settings. At last I googled `git.credential` and realized that `credential managers` is a git concept and `git.credential` is how you config the credential manager of your choice much like your merge/diff tool. Here is [a great post about GCM](https://github.blog/2020-07-02-git-credential-manager-core-building-a-universal-authentication-experience/) - my mistake was to just skim it the first time and discarding it as a MSFT tool that I really didn't need.

Running `git config --list --show-scope` revealed that I had this config setting (with an `unknown` scope?!):

![image](img/git-config-list.png)

I'm guessing that is why git could auth against the repos I could still access.

### Installing git-credential-manager & configuring git.credential

To bring the magic back I had to do these steps:

- [Install `git-credential-manager`](https://github.com/git-ecosystem/git-credential-manager/blob/release/docs/install.md)
- Add it to $PATH (in `~/.zshrc`)..
- ..because the installer added the path to $PATH using `~` instead of `$HOME`

And then add the following to `git config --global`:

```
[credential]
  helper = manager
  credentialStore = keychain
  useHttpPath = true
```

`useHttpPath` is needed [for reasons stated here](https://git-scm.com/docs/gitcredentials#Documentation/gitcredentials.txt-useHttpPath).

After making these changes I got the Azure DevOps login UI when `git fetch`ing and after logging in I was promted to grant access to KeyChain and after that there was a `dev.azure.com` entry in KeyChain.

## Still magical

I still don't know how I got entries into KeyChaing before installning `git-credential-manager` but I'm guessing it was bundled with Visual Studio for Mac (which I have recently uninstalled).
