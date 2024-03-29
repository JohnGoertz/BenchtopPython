# 00.2: Git(hub) for Benchtop Scientists

> _"My code was working but then I changed something and now it doesn't and I don't know how to get it back to working again!" - Every. Coder. Ever._

Git is the answer to this inevitable dilemma. Git and Github are powerful tools to help you keep track of the changes you make to your code, sync your code to the cloud so you and others can access it anywhere, and incorporate changes proposed by collaborators. Intended for developers writing open-source software, it can be difficult to translate these tools into a workflow adapted to experimental science. Here I'll describe my own system for doing just that within the directory structure I describe above.

First, I'll describe how to use git locally (on your computer, not the cloud) as a sort of lab notebook for your code. Next I'll talk about how to use the cloud service *Github* to host your code, keeping it private to yourself and a few collaborators or open for the whole world to see. Next, I'll show you how to use git "subtrees" to re-use utility code in a more elegant way than copy-pasting for every experiment. Finally, I'll work through a couple extra tools that make tracking binary files (e.g., Excel or Word documents, images) or Jupyter Notebooks easier to do. There's a lot here, so don't feel the need to go all-in all at once; you should be able to incorporate each successive section into your workflow individually until you're comfortable with it.

## Motivation

So you have your experiment folders set up like I described in Part 1. Now, how do you save yourself from breaking your code? How do reuse your own code or someone else's? Copy-pasting works for a while, but how do you know where the most up-to-date version is? What if you're working on two experiments at the same time and modifying some utility code for both, how do you reconcile the changes with each other? 

This guide will give you a framework for all that while keeping a few "rules" in mind:
* Everything needed to reproduce experimental analysis should be in one granular folder
* Supporting functions and utility code should remain in the state they were when the analysis was "completed"
* There should be one "master" version of all utility code
* It should be clear which experiments led to changes in to the utility code

## TL;DR

Here's my framework in a nutshell, I'll go more into depth on each part in the rest of this guide.

Let's say you have your Electronic Lab Notebook folder setup like this, with your master utility code folder next to experiments:

```
ELN
├── JG027
│   ├── Data
│   └── Code
│       ├── processJG027.ipynb
│       └── Utils
│           ├── myUtils.py
│           └── processQuantStudio.py
├── JG026
├── JG025
├── ...
└── PythonUtils
    ├── myUtils.py
    └── processQuantStudio.py
```

I'll use `myeln` to navigate to this ELN folder. See part 1 for how to set up this alias in bash.

A separate folder/repo holds the "master" version of the utility code, potentially published on Github for convenience:
 * `myeln`
 * `cd PythonUtils`
 * `git init`

Now upload this to Github if you want.
 
Each experiment is its own repo, located in the parent folder for that experiment (e.g., JG027):
 * `myeln`
 * `cd JG027`
 * `git init`

Git subtree is used to clone the utility repo into the Code folder for each experiment:
 * `git remote add utils https://github.com/JohnGoertz/PythonUtils`
   * alternatively, replace the url with the path to the "master" utilities repo)
   * `git remote add utils ../PythonUtils`
 * `git subtree add utils master --prefix=Code/Utils/ --squash`

When you want to, update this version of the utility code stored in the experiment with the "master" version:
 * `git subtree pull --prefix=Code/Utils utils master --squash`

Commit changes to either experiment-specific code or utility code as normal, but do __not__ commit changes both at the same time.

When you want to update the "master" version of your utility code, push the changes to a branch named for the current experiment, in this case "JG027":
   * `git subtree push --prefix=Code/Utils utils JG027 --squash`

When you're "done" with the experiment, merge this branch into master:
 * `myeln`
 * `cd PythonUtils`
 * `git pull origin JG027` (to grab the new branch)
 * `git pull origin master` (to make sure the master branch is up-to-date)
 * `git merge JG027 --no-ff`
   * I like the "no fast forward" option because it lets me see clearly which changes correspond to which experiment
   
Tag the merge commit with the name of the experiment:
 * `git tag JG027`

Push the updated repo to github. Be sure to include the tags.
 * `git push origin --tags`
 * You can omit the `--tags` flag every time if you run `git config --global push.followTags true` once

Use git Large File System to track your PNG figures in the Results folder.
 * Run these commands once per experiment:
   * `git lfs track "*.png"`
   * `git add .gitattributes`
 * and this every time you want to commit a new version of all your figures at once:
   * `git add Results/*.png`

***
## Version Control: A Lab Notebook for Code

In our experimental lab notebooks, we write down all the steps that allowed each experiment to come together into a finished project. The document becomes a record of the decisions we made along the way, when we changed course and when we needed to double-back because we set off in the wrong direction. Version Control Systems (VCS) like git allow us to do the same for our code. Think of git as a sort of "track changes" for your code. Each time you've completed a significant change to your code, you "commit" that change to make a new version with a brief description of what you did. Git keeps track of all these versions and the differences between them, allowing you to view a summary of your changes over time. If you've realized you made a mistake, you can easily revert your code to what it look like at one of your previous commits, back when you *know* it was working.

### Getting set up

If you're on a Linux or Mac machine, you already have git installed. Just open a terminal, navigate to your experiment folder, and you're good to go. For Windows you'll have to download and install it. In either case, you can choose between using the bash terminal (command line) or using the Github Desktop graphical interface. I'd recommend using the terminal; some of the tools I lay out here (like subtree and LFS) I'm really not sure how to do with Github Desktop. It might seem daunting at first, but once you get a hang of navigating your file system (namely using the commands `cd` and `ls`) there are really only five to ten commands you'll use within git.

I'll only quickly go over how to use git in the context of my organization system. There are much more comprehensive guides out there. A few that I'd recommend:
* __[A Friendly Intro to Git and Github](https://kirstiejane.github.io/friendly-github-intro/)__
* __[An Interactive Git Tutorial with Katacoda](https://www.katacoda.com/courses/git)__
 * __[Codecademy](https://www.codecademy.com/learn/learn-git)__ used to have a free tutorial, but it seems to be pro-only now...
* __[Github's own guide](https://guides.github.com/activities/hello-world/)__
* __[Git cheatsheet - "no deep shit"](https://rogerdudler.github.io/git-guide/)__
* __[Reproducible Research in R](https://cambiotraining.github.io/reproducibility-training/)__

### Git Repos: Writing The Notebook Itself

A git repository (or *repo*) is collection of folders and files you're keeping track of. Let's start by tracking an experiment. 
* Go to a folder set up like *JG027* above (I'll call sometimes call this the parent folder), then type `git init`. Boom. You've made your first git repo.
* Add a file to the "Code" folder called "MolecFightClubRules.txt" (if you're in the terminal you can do this with something like `echo "1. Talk about Molecular Fight Club" > Code/MolecFightClubRules.txt`)
* Type `git status`. You'll see that git has found a new "untracked" file
* "Stage" the new file with `git add Code/FightClubRules.txt`
* "Commit" the file, and give it a message to help you remember what you did, with something like `git commit -m "Create Fight Club Rules, add first rule"`
 * Convention dictates these are short, imperative statements, e.g. "add/change/fix new thing"
* Let's do it all again!
 * Add a new line to the same file, something like `echo "2. Talk about Molecular Fight Club constantly" >> Code/MolecFightClubRules.txt`
 * `git add Code/FightClubRules.txt`
 * `git commit -m "Add second rule"`
* Take a look at all the progress you've made with `git log`
 * You can get a prettier, more concise view with `git log --all --decorate --oneline --graph`, just remember the mnemonic "A DOG"

That's pretty much it, that's the basics of git. Now, you won't be committing every line individually, but you should commit often. It's good to keep your commits "atomic", meaning each contains a set of complete but self-contained changes. Maybe you added a new function, and you tested it and it works. Commit. Process the results for one of your samples into a figure? Commit. Make that loop through all your samples to plot in the same figure? Commit. You get the idea. Any time you find yourself thinking "Okay, if I mess the next thing up, I at least want to get back to this point", commit.

So now for how you get back to that point. Did adding that second line break something, and you want to go back to an earlier commit? Look at the git log and find the seven-digit alphanumeric at the beginning of the line for the commit you want to go back to. This is called the *hash*, and it's git's name for that commit. Let's glance at what the code looked like at that point by running `git checkout 70fd0b1` (or whatever your first commit is called). Open up the file and you'll see the second line is now gone. We haven't erased that second commit, though. If you run the `git log` command again you'll see the commit is still there, but you're currently looking at the state of your code at the first commit (indicated by the `HEAD`). To do completely erase the second commit, first go back to most recent commit with `git checkout master` (or replace "master" with whatever branch you're on - more on branches below). Now type `git reset --hard 70fd0b1`, this will revert the code to its state at that point, clear all the commits between now and then, and also get rid of any *uncommitted* changes you've made. Your whole repository now looks exactly like it did then, and there's no going back.

You can add multiple files at this point, for instance if you made interrelated changes to both your "processJG027" and "myUtils" files, but as a general rule I try to commit changes to one file at a time. I also try to start the commit message with a short indication of which file I changed, since that's not immediately obvious from the log; so if I made a change to "processQuantStudio.py" my commit message might be something like "pQS: fit logistic curve".

Just remember, commit often. Your future self will thank you.

### Branching: A Safe Space to Experiment

So you're coding along happily, committing frequently, when you come to a crossroads. Your code works great right now, but maybe you want to reorganize and clean it up (this is called *refactoring*, and it's a good idea to do so periodically for code you reuse frequently), perhaps breaking big chunks down into more modular functions. Or maybe you need to change things a bit to make it work for experiment B but need it to work exactly as it does now for experiment A. This is where *branching* comes in.



## Remote Repositories: Sharing is Caring

### Binder: Turning Your Code Into A Public Playground 

## Subtrees: Recycling Code

## Large File System: Tracking Raw Data

Git works best on *text* files. All Python, R, and Matlab code is really just text files, as are CSV files. This is because each time you commit a change, git essentially saves a full copy of that file. For text files this leads to minimal overhead on your hard drive, but if you update figure PNG files frequently that could add up quickly. If you want to include large, *binary* files (e.g., .xlsx, .png) in the git tracking (and/or want them available on Github), use __[Git Large File System](https://git-lfs.github.com/)__ (LFS).

Once it's installed, using LFS for all PNG images, for instance, is as simple as `git lfs track "*.png"` followed by `git add .gitattributes`. You only need to do this once per repo, so I usually do this right after initializing. You can then add/commit the individual files as usual. You can find a deeper guide __[here](https://www.git-tower.com/learn/git/ebook/en/command-line/advanced-topics/git-lfs)__ or __[here](https://www.atlassian.com/git/tutorials/git-lfs)__. Note, to track both modern and legacy Excel files (.xls and .xlsx), use `git lfs track "*.xls*"`.

## Git and Jupyter Notebooks: Filters

# Summary and Cheatsheets

