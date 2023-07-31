---
title: Git Rev News Edition 101 (July 26th, 2023)
layout: default
date: 2023-07-26 12:06:51 +0100
author: chriscool
categories: [news]
navbar: false
---

## Git Rev News: Edition 101 (July 26th, 2023)

Welcome to the 101st edition of [Git Rev News](https://git.github.io/rev_news/rev_news/),
a digest of all things Git. For our goals, the archives, the way we work, and how to contribute or to
subscribe, see [the Git Rev News page](https://git.github.io/rev_news/rev_news/) on [git.github.io](http://git.github.io).

This edition covers what happened during the months of June 2023 and July 2023.

## Discussions

<!---
### General
-->

### Reviews

+ [[RFC PATCH 0/8] Introduce Git Standard Library](https://public-inbox.org/git/20230627195251.1973421-1-calvinwan@google.com/)

  Calvin Wan sent a 8 patch long RFC series to the mailing list to
  introduce a "Git Standard Library". This new library was intended to
  be a base library for other Git libraries to build upon.

  This followed previous effort from Calvin and Elijah Newren to
  cleanup [headers](https://lore.kernel.org/git/pull.1525.v3.git.1684218848.gitgitgadget@gmail.com/)
  and internal APIs like the
  [`strbuf` API](https://lore.kernel.org/git/20230606194720.2053551-1-calvinwan@google.com/)
  to manipulate character strings and the
  [git-compat-util API](https://lore.kernel.org/git/20230606170711.912972-1-calvinwan@google.com/)
  to provide wrapper compatibility functions masking OS differences.
  It also followed previous
  [discussions about turning parts of Git into libraries](https://lore.kernel.org/git/CAJoAoZ=Cig_kLocxKGax31sU7Xe4==BGzC__Bg2_pr7krNq6MA@mail.gmail.com/)
  and the ongoing
  [video conferences](https://lore.kernel.org/git/CAJoAoZmBFTi5SFRuG8uh4ZyGs7pKQTYQLzZAC82zh2pMSggX3A@mail.gmail.com/)
  that are regularly hosted by Google about libifying Git.

  Building Git already involves creating an internal library called
  `libgit.a` that contains a lot of common code used by many Git
  commands. The Git executable is then created by linking all the
  object files for the Git subcommands (like `git log`) and the `git`
  command itself against `libgit.a` and a few external
  dependencies. The goal with the new Git Standard Library, also
  called `git-std-lib.a`, and the Git libification effort is to have a
  number of small independent libraries (like `object-store.a`, and
  `config.a`) all using `git-std-lib.a`. These small independent
  libraries would also be linked together to form `libgit.a`, which
  would then be used to create the Git executable like today. The
  benefit would be that "if someone wanted their own custom build of
  Git with their own custom implementation of the object store, they
  would only have to swap out `object-store.a` rather than do a hard
  fork of Git".

  The new Git Standard Library is considered necessary for the
  libification effort, because there are numerous circular and
  ubiquitous dependencies that are very difficult to untangle and it
  probably wouldn't be worth untangling them. As the libification
  effort doesn't promise stability of interfaces though, it would
  still be possible to extract some small libraries from
  `git-std-lib.a` later if there is ever a need to be able to swap
  them out.

  Calvin noted that a pitfall of this series was the introduction of
  `#ifdef GIT_STD_LIB` preprocessor instructions to stub out some code
  and hide some function headers. He asked for comments about how it
  would be possible to avoid those instructions which make the code
  harder to understand. He also mentioned that some of the
  compatibility code in `compat/` still had dependencies outside
  `git-std-lib.a`, but that it just meant that "some minor
  compatibility work" might be needed in the future.

  As to the testing of the new library, he said that "the temporary test file
  added by the series will be replaced with unit tests once a unit
  testing framework is decided upon", pointing to the
  [related discussion](https://lore.kernel.org/git/8afdb215d7e10ca16a2ce8226b4127b3d8a2d971.1686352386.git.steadmon@google.com/).
  That discussion is actually an RFC patch series that in its latest
  version only adds an
  [asciidoc document](https://github.com/steadmon/git/blob/unit-tests-asciidoc/Documentation/technical/unit-tests.adoc)
  explaining the plan to add a unit test framework to Git and
  containing a table comparing the possible unit test frameworks that
  Git could adopt.

  Victoria Dye and then Jeff Hostetler replied to the first patch in
  Calvin's series saying that they didn't agree with the fact that the
  Trace2 code wouldn't be in `git-std-lib.a`. They pointed out that it
  should be possible to use the Trace2 tracing functions everywhere in
  the Git code, even in low-level functions. Calvin replied that he
  would look into possible solutions like redrawing the boundaries of
  the library or stubbing out tracing in it to accommodate that need.

  Phillip Wood commented on a few patches saying he liked the idea,
  but suggested that the library should also contain the code related
  to [gettext](https://www.gnu.org/software/gettext/) and to some
  basic data structures ("hashmap.c" and "string-list.c"). He also
  suggested some improvements and a way to deal with the Trace2 issue.

  Junio Hamano, the Git maintainer, was happy that one patch removed
  the dependency many files had on "config.c". He wondered though if
  another patch that removed a wrapper function to remove a dependency
  did the right thing, as the function had a number of callers. Glen
  Choo then chimed in to suggest an alternative way to remove that
  dependency and both Calvin and Junio agreed that this would be a
  good way forward.

  Glen separately reported that he had trouble building the series.
  Calvin clarified that the series was based on Git v2.41 and sent a
  [link to the code](https://github.com/calvin-wan-google/git/tree/git-std-lib-rfc).

  Linus Arver also replied to Calvin's series by asking a number of
  questions. He wondered if adding the standard library would make it
  harder or not to refactor code into separate libraries or to create
  separate programs that would use only some Git API, like for example
  the API in "trailer.c" that helps parse commit message trailers
  (e.g. "Signed-off-by: Author Name <author@example.com>). He asked
  for more information about the tradeoffs between accepting the
  circular dependencies and untangling them, and made a number of
  other suggestions.

  It's not clear yet if the libification effort and the idea of a Git
  Standard Library will bring a lot of changes to the code base soon,
  but that could be an interesting possibility.

<!---
### Support
-->


## Developer Spotlight: Martin Ågren

* Who are you and what do you do?

  I'm Martin Ågren. I live in Sweden, where I spend some of my spare time
  in the garden or tending to the bees to the best of my abilities. I also
  enjoy reading books and listening to music (not at the same time).

* What would you name your most important contribution to Git?

  The number of breathtaking features I've contributed is probably a
  one-digit number ending in a zero. That said, I think I've contributed a
  bit to the documentation by fixing some fairly ugly misrenderings, but
  also by aligning the way it's formatted by the two tools we support,
  asciidoc and asciidoctor. I guess we'll never know this for a fact, but
  there's a chance that I've saved someone's crontab by fixing a bug that
  would eat it. I'm very happy that I fixed that bug before it was ever
  included in a release.

* What are you doing on the Git project these days, and why?

  I'm mostly just tinkering. I very rarely feel like there's something
  actually missing from Git. I'm mostly trying to contribute in order to
  show that gratitude and to help others, without occupying too much
  bandwidth.

* If you could get a team of expert developers to work full time on
  something in Git for a full year, what would it be?

  Complete the hash function transition. brian m. carlson has done a
  tremendous job making sure there are these two parallel worlds, if you
  want. What's missing now is making them interoperable. This is not
  necessarily the biggest *problem* in current Git, but it could be
  something that won't be fixed by short-term, this-quarter,
  profit-maximizing actors, so if I could decree a team to work on that
  without having to worry about "return on investment" and such, I'd
  probably go for that.

* If you could provide users of Git with one piece of advise,
  what would it be?

  The one thought I would like to somehow convey to everyone using Git is
  to commit early, commit often. Whatever crap you have ever had in your
  working tree, there's an object containing it. Use `git reset --hard`,
  `git rebase`, `git cherry-pick`, whatever floats your boat, you will be
  able to bring it back and polish it up into a Git history that looks
  like you knew what you were doing all along.

  I think this is really the point about Git: it teaches you how to
  pretend to be a good programmer, and once you start thinking of shaping
  your work like that, you actually might turn into one. Not because "fake
  it till you make it", that's just bullshit, but because you actually
  spend time approaching problems the right way and start thinking about
  how you present your solution.

  Your "solution" is then not just the state of the working tree ("look!
  it compiles and all the tests pass, so it must be good!"), but also how
  you got there, as a series of well-motivated incremental changes.

* What is your favorite Git-related tool/library, outside of
  Git itself?

  I'm a big fan of `tig`, especially `tig blame`. I simply never use `git
  blame`.  If I'm allowed to count `git/contrib` as "outside of Git
  itself", I'll be more than happy to recommend `git jump`. It's not
  especially sexy, but I probably use it every single day and I find it
  extremely helpful.

* Do you happen to have any memorable experience w.r.t. contributing to
  the Git project? If yes, could you share it with us?

  I still sometimes think back to when I posted my first patch series to
  the list. Peff wrote "[...] I'm very impressed with the attention to
  detail for a first-time contributor.", to which Junio replied "Yes.".
  Of course, a part of even remembering that is vanity on my part, but I
  do think those two sentences are fairly representative of each of their
  communication styles. They also capture perfectly well the kind of
  review style that I wish a lot more projects used. You know, it is
  allowed to not just point out something that is wrong or could be
  better.

* What is your toolbox for interacting with the mailing list and for
  development of Git?

  For the list, it's gmail.com, lore.kernel.org/git, `git am`, `git format-patch`,
  and `git send-email`. I keep thinking I should set up something
  more advanced, but for the limited volumes I'm handling, it's fine. For
  development of Git [and other stuff], it's Vim, `git diff`, `git add -p`,
  `git commit --amend`, `git rebase -i`, `git range-diff`, `tig blame`,
  `git jump` (grep, merge, diff) and ... maybe that's about it. Well, `git show`
  and `git log` of course. Please note the `-p` in `git add -p`. I would like
  to live in a world where no-one blindly does `git add . && git commit`.

* What is your advice for people who want to start Git development?
  Where and how should they start?

  Do something you enjoy doing. Of all the people born any given year, not
  even one of them, on average, will ever become president of the United
  States.  Don't do open source because it could land you a nice job
  somewhere, sometime.  Don't do Git development because it seems like a
  good investment. Do open source because you believe in it and see some
  random thing that you want to contribute to. If you don't see that,
  plant a flower instead and watch it grow.

  This obviously comes from someone who is privileged enough to be able to
  say "don't worry, be happy" and talk in metaphors about gardening. That
  said, I do think there's a difference in keeping bees and tending to
  them. You shouldn't want to keep them, you should want to help them do
  their thing. And if you want to help Git do its thing, great!

  Start by lurking on the mailing list to get a feel for how it works.
  Then do some small improvement, and avoid growing the scope too much.
  Sleep on your patch, review it yourself and iterate that process a few
  times before actually sending it off.


## Other News

__Various__
- A [Git community](https://programming.dev/c/git) has been created in the programming.dev Lemmy instance.
- [Passwordless authentication is now available on GitHub.com](https://github.blog/2023-07-12-introducing-passwordless-authentication-on-github-com/)
  (with [passkeys](https://www.passkeys.io/)), as a public beta.
- [Meet Smart Locks, a new way to reduce merge conflicts with Unity Version Control](https://blog.unity.com/engine-platform/unity-version-control-smart-locks)
  (previously Plastic SCM).
    + Locks in DVCS were [also supported](https://ericsink.com/vcbe/html/veracity_locks.html)
      in SourceGear's [Veracity SCM](http://veracity-scm.com/)
      (which is no longer developed, it's last version was from 2013).
    + See also [Git for games: current problems and solutions](https://www.youtube.com/watch?v=K3zOhU3NdWA&list=PL0lo9MOBetEFqBue4vNcTEnkBjgIQU1Q3&index=7)
      video from Git Merge 2019.

__Light reading__
+ [Scaling merge-ort across GitHub](https://github.blog/2023-07-27-scaling-merge-ort-across-github/)
  by Matt Cooper on GitHub Blog, describing how and why GitHub have switched
  from performing merges and rebases [using libgit2](https://github.blog/2015-12-15-move-fast/)
  to using merge-ort for it ([available in Git since 2.33](https://github.blog/2021-08-16-highlights-from-git-2-33/#merge-ort-a-new-merge-strategy)),
  and performing worktree-less rebases using the new [work-in-progress](https://public-inbox.org/git/20230407072415.1360068-1-christian.couder@gmail.com/t/#u)
  [`git replay` command](https://github.com/newren/git/commits/replay).
  [Merge commits were created using the merge-ort strategy](https://github.blog/changelog/2022-09-12-merge-commits-now-created-using-the-merge-ort-strategy/)
  on GitHub since September 2022, as mentioned in [Git Rev News Edition #91](https://git.github.io/rev_news/2022/09/30/edition-91/).
+ [Git and Jupyter Notebooks: The Ultimate Guide](https://www.reviewnb.com/git-jupyter-notebook-ultimate-guide)
  by ReviewNB.  The article mentions the following tools:
    + [nbdime](https://nbdime.readthedocs.io/en/latest/),
      mentioned first in [Git Rev News Edition #37](https://git.github.io/rev_news/2018/03/21/edition-37/),
    + [jupyterlab-git](https://github.com/jupyterlab/jupyterlab-git) extension,
      described in more detail in [How to use the JupyterLab Git extension](https://blog.reviewnb.com/jupyterlab-git-extension/)
      on ReviewNB Blog,
    + [ReviewNB](https://www.reviewnb.com/) app for GitHub, for when
      [rich notebook diffs on GitHub](https://github.blog/changelog/2023-03-01-feature-preview-rich-jupyter-notebook-diffs/) fail,
    + [nbdev](https://nbdev.fast.ai/), which includes support for better Git handling for Jupyter notebooks
      (see [Fix merge conflicts in jupyter notebooks](https://nbdev1.fast.ai/merge.html) for nbdev1,
      and [Git-Friendly Jupyter](https://nbdev.fast.ai/tutorials/git_friendly_jupyter.html) for nbdev2),
      and was mentioned in [Git Rev News Edition #69](https://git.github.io/rev_news/2020/11/27/edition-69/),
      and also mentioned in [Edition #100](https://git.github.io/rev_news/2023/06/30/edition-100/)
      as one of alternatives for [databooks](https://databooks.dev/) tool,
    + [NBviewer](https://nbviewer.org/) and [Binder](https://mybinder.org/) services.
+ [How To Automate Your Server Database Backup Using Git](https://confidence.sh/blog/automate-your-server-database-backup/)
  and cron,
  by Confidence Okoghenun on his blog (also on [DEV\.to](https://dev.to/megaconfidence/how-to-automate-your-server-database-backup-using-git-2pc2)).
+ [How to Use Git Shallow Clone for Faster Repository Cloning](https://www.howtogeek.com/devops/how-to-use-git-shallow-clone-for-faster-repository-cloning/)
  (and cloning only a single branch, and creating sparse blobless and treeless clones)
  by Anthony Heddings on How-To Geek.
  This can be useful when you want to examine Git repos with long histories,
  and is also useful for Continuous Integration pipelines (CI/CD),
  since the CI/CD process likely does not need to know the entire Git history.
+ [How To Undo or Reverse a Git Merge](https://www.howtogeek.com/devops/how-to-reverse-a-git-merge/)
  by Anthony Heddings on How-To Geek.
+ [The Git Rebase Handbook – A Definitive Guide to Rebasing](https://www.freecodecamp.org/news/git-rebase-handbook/)
  by Omer Rosenbaum on freeCodeCamp,
  continuing from [Git Merge – The Definitive Guide](https://www.freecodecamp.org/news/the-definitive-guide-to-git-merge/)
  mentioned in [Git Rev News Edition #99](https://git.github.io/rev_news/2023/05/31/edition-99/)
  (and other posts).
+ [Mastering Git Shortcuts: A Guide to Git Aliases](https://dev.to/pradumnasaraf/mastering-git-shortcuts-a-guide-to-git-aliases-324j)
  by Pradumna Saraf on DEV\.to - though the guide is limited to simple aliases,
  and does not cover forcing an alias to be treated as a shell command,
  or tricks that one can use for advanced handling of alias parameters.
+ [.gitattributes Best Practices](https://rehansaeed.com/gitattributes-best-practices/)
  by Muhammad Rehan Saeed on his blog (2020).
+ [The Power of Git: A Guide to Collaborative Version Control](https://dev.to/opensauced/the-power-of-git-a-guide-to-collaborative-version-control-dl6)
  by BekahHW for OpenSauced on DEV\.to.
+ [A Guide to Perfecting Pull Requests](https://dev.to/karaluton/a-guide-to-perfecting-pull-requests-2b66)
  by Kara Luton on DEV\.to.
+ [Effective Code Reviews](https://addyosmani.com/blog/code-reviews/)
  by Addy Osmani on his blog.
+ [The Gentle Art Of Patch Review](https://sage.thesharps.us/2014/09/01/the-gentle-art-of-patch-review/)
  by Sage Sharp on their blog (2014).


__Easy listening__
+ [Git with Derrick Stolee](https://podrocket.logrocket.com/git-scalar)
  on PodRocket (a web development podcast from LogRocket).


__Git tools and sites__
+ [Emoji-Log](https://github.com/ahmadawais/Emoji-Log) — A simple Emoji Git commit log messages spec standard.
    + Similar [GitMoji](https://gitmoji.dev/) standard — an emoji guide for your commit messages,
      was mentioned in [Git Rev News Edition #47](https://git.github.io/rev_news/2019/01/23/edition-47/).
+ [thefuck](https://github.com/nvbn/thefuck)
  is a command line application
  which corrects your previous console command.
  Among others it includes series of rules for correcting mistakes when using Git
  (like typos in command names, trying to `git add` an ignored file, etc.).
  Written in Python.
+ New Relic [CodeStream](https://www.codestream.com/) is a free [open-source](https://github.com/TeamCodeStream/codestream)
  extension for VS Code, Visual Studio, and JetBrains
  (with support for more IDEs on the way)
  for putting collaboration tools in the IDE:
  supporting pull requests, issue management, observability, and code discussion.
+ [jupyterlab-git](https://github.com/jupyterlab/jupyterlab-git)
  is a JupyterLab extension for version control using Git.
+ [ReviewNB](https://www.reviewnb.com/) is a paid GitHub and Bitbucket app
  for rich diffs and commenting (including code review) for Jupyter Notebooks,
  free for open source (for public repositories) and academic use:
  see [pricing](https://www.reviewnb.com/#pricing).
  You can request self hosted installation (distributed as Docker image).
+ [pwclient](https://github.com/getpatchwork/pwclient) is the command-line client for 
  the [Patchwork](http://jk.ozlabs.org/projects/patchwork/) web-based patch tracking tool.
  Patchwork was mentioned in [Git Rev News Edition #20](https://git.github.io/rev_news/2016/10/19/edition-20/),
  in passing in [Edition #48](https://git.github.io/rev_news/2019/02/27/edition-48/)
  (as then base for [snowpatch](https://github.com/ruscur/snowpatch) CI for patches,
  now also supporting GitHub Actions),
  in passing in [Edition #55](https://git.github.io/rev_news/2019/09/25/edition-55/)
  (as being similar to [patchew](https://github.com/patchew-project/patchew)
  patch tracking and testing system),
  and in passing in [Edition #88](https://git.github.io/rev_news/2022/06/30/edition-88/)
  (as being used by [Sourceware](https://sourceware.org/) service).
  Patchwork is used for [some Linux kernel development](https://patchwork.kernel.org/).
+ JetBrains Academy [offers a 'Master Git and GitHub in 5 Hours'](https://blog.jetbrains.com/education/2023/07/06/introduction-to-git-track/)
  course [on Hyperskill](https://hyperskill.org/tracks/48) for free,
  with the project-based part of the course available only for paid users (Premium).

## Releases

+ Git for Windows [2.41.0(3)](https://github.com/git-for-windows/git/releases/tag/v2.41.0.windows.3),
[2.41.0(2)](https://github.com/git-for-windows/git/releases/tag/v2.41.0.windows.2)
+ libgit2 [1.7.0](https://github.com/libgit2/libgit2/releases/tag/v1.7.0)
+ GitLab [15.11.13](https://about.gitlab.com/releases/2023/07/27/gitlab-15-11-13-released/)
[16.2.1](https://about.gitlab.com/releases/2023/07/25/gitlab-16-2-1-released/),
[16.2](https://about.gitlab.com/releases/2023/07/22/gitlab-16-2-released/),
[15.11.12](https://about.gitlab.com/releases/2023/07/17/gitlab-15-11-12-released/),
[16.1.2, 16.0.7, and 15.11.11](https://about.gitlab.com/releases/2023/07/05/security-release-gitlab-16-1-2-released/),
[16.1.1, 16.0.6, and 15.11.10](https://about.gitlab.com/releases/2023/06/29/security-release-gitlab-16-1-1-released/)
+ Bitbucket Server [8.12](https://confluence.atlassian.com/bitbucketserver/bitbucket-server-release-notes-872139866.html)
+ Gerrit Code Review [3.6.6](https://www.gerritcodereview.com/3.6.html#366),
[3.7.4](https://www.gerritcodereview.com/3.7.html#374),
[3.8.1](https://www.gerritcodereview.com/3.8.html#381)
+ GitHub Enterprise [3.9.1](https://help.github.com/enterprise-server@3.9/admin/release-notes#3.9.1),
[3.8.6](https://help.github.com/enterprise-server@3.8/admin/release-notes#3.8.6),
[3.7.13](https://help.github.com/enterprise-server@3.7/admin/release-notes#3.7.13),
[3.6.16](https://help.github.com/enterprise-server@3.6/admin/release-notes#3.6.16),
[3.9.0](https://help.github.com/enterprise-server@3.9/admin/release-notes#3.9.0)
+ GitKraken [9.6.0](https://help.gitkraken.com/gitkraken-client/current/)
+ GitHub Desktop [3.2.7](https://desktop.github.com/release-notes/)
+ Tower for Mac [10.1](https://www.git-tower.com/release-notes/mac?show_tab=release-notes)
+ git-credential-oauth [0.9.1](https://github.com/hickford/git-credential-oauth/releases/tag/v0.9.1),
[0.9.0](https://github.com/hickford/git-credential-oauth/releases/tag/v0.9.0)
+ gitolite [v3.6.13](https://groups.google.com/g/gitolite-announce/c/Ba5yVnkf_lQ/m/xB72ygAIDQAJ)


## Credits

This edition of Git Rev News was curated by
Christian Couder &lt;<christian.couder@gmail.com>&gt;,
Jakub Narębski &lt;<jnareb@gmail.com>&gt;,
Markus Jansen &lt;<mja@jansen-preisler.de>&gt; and
Kaartic Sivaraam &lt;<kaartic.sivaraam@gmail.com>&gt;
with help from Eren Canpolat and Bruno Brito.
