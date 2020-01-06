---
title: "Smaller Git Repo Size Using Git LFS<br> (🤔 Xcode Assets)"
layout: post
date: 2020-1-7
image: /assets/images/posts/git-lfs/logo.png
headerImage: true
tag:
- Git
- Large File Storage
- Asset management
- Xcode
star: false 
category: blog
author: jasonzurita 
description: Smaller Repo Size Using Git Large File Storage (LFS)

---

# Summary

<sub>**If you want to see a comparison of Git with and without Git LFS, see [this section below](#comparison-of-git-with-and-without-git-lfs).**</sub>

Have you ever gone to clone a repository (repo), and ended up just sitting there staring at the slow moving percentage...😶?

Yeah, me too — this may help!

Often times with a project, like with Xcode projects, we check assets into Git so that our app has immediate access to those files (images, videos, audio files, etc.). Over time, changes to those assets may end up bloating your repo's size<sup>1</sup> 😅. When this happens, Git operations like cloning can take an unfortunate amount of time...

Luckily, there is a Git extension called [Git Large File Storage (LFS)](https://git-lfs.github.com) to help manage your large files (without changing your Git workflow 😁), which means:
- 🏃‍♀️ Faster Git cloning and fetching since there is less data to download!
- 💪 Reduce the size of your repo.

<sub>1. Git is less effective at compressing binary files, and therefore ends up storing near duplicates of those files. [Read more](https://stackoverflow.com/a/21082870/6713525).</sub>


---


{: #initial-git-lfs-setup}
# Getting Started with Git Large File Storage (LFS)
There are different ways to install [Git LFS](https://git-lfs.github.com), but if you are on a Mac: `brew install git-lfs`

Setup is pretty straight forward. Do the following to add files or folders to Git LFS:
{: #installing-git-lfs}
- Setup your repo to use Git LFS.
  + `git lfs install`
- Add a file type to Git LFS.
  + `git lfs track "*.png"`
    + Note: replace _png_ with other file type(s).
- Add a whole folder to Git LFS.
  + `git lfs track path/to/some/folders/*`
    + If you have an Xcode project, consider adding the assets folder (or if you have a custom resource folder). Look for the _Assets.xcassets_ folder 😉.
- Make sure you commit the _.gitattributes_ file.
  + `git add .gitattributes && git commit -m "Add Git LFS attribute dot file" && git push`
- For files that are already being tracked in Git.
  + `git lfs migrate import --no-rewrite test.zip *.mp3 *.psd`

_Don't forget to make sure that [installing Git LFS](#installing-git-lfs) is part of your setup README for project :).<br>See the [first bullet](#considerations) under considerations below for more details._

That's it 🎊 — Go about your normal Git workflow like before, and the _marked_ files will now be managed by Git LFS!


## Existing Files - For The Maximum Benefit
{: #force-push-warning}
> **Caution:** The below steps result in a <u>destructive git repo operation, and should be done carefully</u> After performing the below steps, all members of your team will need to delete their local repos and re-clone the project repo. This is because their existing local repos will no longer be compatible with the newly _force pushed master_.

For an existing repo, you will get the biggest gain from using Git LFS by retroactively applying Git LFS to files and folders. Here is how to integrate Git LFS as if you have been using it since the beginning of your project.

For setup, do the same [steps as before](#initial-git-lfs-setup), but for files that are already being tracked in Git:
- Migrate file type(s) that you want.
  + `git lfs migrate import --include-ref=master --include="*.png"`
- Migrate the folder(s) that you want.
  + `git lfs migrate import --include-ref=master --include="path/to/some/folder/*"`
    + If at one point this folder's path changed, you are better off tracking specific file types. This is because Git LFS migrate will go back as far as it can, but stops when the path name changed.
- Make sure your project still works 😅.
- Force push (e.g., master).
  + git push master \-\-force
    + **Careful — [See caution above!](#force-push-warning)**


---


# Comparison of Git with (and without) Git LFS

As an example I created an example project ([https://github.com/jasonzurita/Git-LFS-Test](https://github.com/jasonzurita/Git-LFS-Test)) to compare Git with and without Git LFS.
- I created a large-ish _dummy_ movie file.
- Then, I created several small change commits to that movie file like trimming the movie by a few seconds.
- Then to make the comparison, I created a branch off of master and retroactively used Git LFS. The branch is named [_with-git-lfs_](https://github.com/jasonzurita/Git-LFS-Test/tree/with-git-lfs).

**The result was that cloning the project took close to <u>half the time (~15s vs ~30s)</u> when directly cloning the branch using Git LFS — and this is for a small example project. Imagine the potential impact for more mature projects with lots of file changes of this nature!!**

To try this yourself:
- Clone master (and time how long that takes).
  + `git clone https://github.com/jasonzurita/Git-LFS-Test.git test-master`
- Clone the branch with Git LFS setup (and time how long it takes).
- `git clone --single-branch --branch with-git-lfs https://github.com/jasonzurita/Git-LFS-Test.git test-lfs`


You can also see the difference Git LFS makes in the Git repo size here:
<div style="text-align:center"><img src="/assets/images/posts/git-lfs/repo-size-comparison.png" alt="A comparison of repo size between regular git and git lfs"/></div>


---


# Considerations
- When cloning a repo with Git LFS, you will need to make sure Git LFS is installed or you may get some errors when cloning the repo. Furthermore, best to add this additional setup step to the _README_ or setup script if you have one.
- If you have a CI/CD system setup, make sure you also add the installation of Git LFS to that workflow.
- Each git service is different in their support for Git LFS, so be sure to check them out.
  + For example: [GitHub's docs](https://help.github.com/en/github/managing-large-files/configuring-git-large-file-storage), [BitBucket's docs](https://confluence.atlassian.com/bitbucket/git-large-file-storage-in-bitbucket-829078514.html)
- Although you may not have to address this, there are some [GitHub pricing implications worth checking out](https://help.github.com/en/github/setting-up-and-managing-billing-and-payments-on-github/managing-billing-for-git-large-file-storage).
- For more information on Git LFS: `git lfs --help`
- Git LFS is [open sourced here](https://github.com/git-lfs/git-lfs).

---

Thank you for reading!

Feel free to reach out on [Twitter](https://twitter.com/jasonalexzurita) — cheers!
