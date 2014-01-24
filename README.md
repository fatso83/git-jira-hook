=======================================================================
Copyright 2009 Broadcom Corporation

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

   http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
========================================================================


                    git-jira-hook README
                    ********************

                    Author: Joyjit Nath
                    *******************


Table of Contents
=================
    1. Introduction
    2. System Requirements
    3. Installation
    4. Using git-jira-hook
    5. Known limitations
    6. Frequently Asked Questions (FAQ)
    7. Credits


1. Introduction
===============

1.1 Get git and Jira to work in Harmony
---------------------------------------
If you are using git [1] for source control and Jira [2] for bug 
tracking, then the git-jira-hook script might be useful to you.

Once you have the script setup in your environment, every time you 
make a git commit, a comment is automatically posted to an open 
issue in Jira.

This script will also "enforce" that for every git commit, you have at
least one Jira issue that you are referencing.

This way you have a paper trail of the history behind each and every
git commit.

This is particularly useful for corporate git repositories.


1.2 Example use
---------------
In order to specify which issue (or issues) you want the commit message 
to get tracked to in Jira, you place magic text markers such as:

 "refs #NNN"  or "fixes #NNN" 

anywhere in your commit message, where "NNN" is the name of an open 
Jira issue.

For example, say you have typed in the following commit message in
your git repository.

    Hey look! This is my very first git-commit

    Using the new and fresh git-jira-hook

    refs #SW-189, refs #HW-278 fixes #FW-702


The following things will happen:

In your Jira bug database, for projects named "TST", "HW" and "FW"
in Jira, for issue numbers "SW-189", "HW-278", and "FW-702", the 
following comments will be added:

    commit 424daa955f5c8a17aab9d524071f65f1999769a9
    Author: Joyjit Nath <joyjit@mycompany.com>
    Date: Tue Aug 25 15:12:39 2009 -0700

    Hey look! This is my very first git-commit

    Using the new and fresh git-jira-hook

    refs #SW-189, refs #HW-278 fixes #FW-702

In addition, 
    * the issue "FW=702" will be marked as resolved. (NOTE: 
      this "resolved" part does not work yet. See "Known 
      Limitations" section.)

1.3 Wait! there's  more
-----------------------

If your git repository is exposed using gitweb [3], an hyperlink
linking to the exact commit will also be embedded in the Jira issue
comment that was added. This enables anyone to examine your git
commit simply by clicking on the hyperlink.

2. System Requirements
======================
    - Python 2.x and python modules: SOAPpy, ConfigParser.
      I have tested with Python 2.5.2.

    - A Jira installation with Remote APIs enabled.

    - git version 1.6.x.y (I have tested with 1.6.0.4).

    - Linux or some other similar Unix flavor (I have tested with 
      CentOS 5.x).

    - OPTIONAL, but Highly recommended: gitweb [4] which has been 
      setup with "upstream" git repositories.



3. Installation
===============
Here is a typical example of how this hook may be used.

In a corporate setting where git is used, there is typically an 
"upstream" or "public" repository. And then developers have their 
"private" repositories.  For their day-to-day work, the developers 
use their private repositories.  Periodically, they (either 
directly, or via gatekeepers) push changes from their private 
repository to the "upstream" one. Also, the "upstream" repository 
is typically a bare repository, and no actual commits are done
here.

In the private repository, the hook should be installed, but only 
*partically*. Every time a commit is made, the hook only checks to
make sure that the commit text conforms to correct formatting (i.e. 
the magic references to Jira issues are present). No Jira issues 
are updated.

The hook is completely installed in the "upstream" repository.

Whenever a commit is made in the upstream repository or a "git push" 
is done to it, the  installed hook will kick in and validate the 
commit message, followed by update of the Jira issue.


3.1 Installtion for "private" repository
----------------------------------------
(i)   Copy this script to <your-git-repo-GIT-dir>/hooks/commit-msg 
      and mark it executable
      Example:
       cp git-jira-hook joyjit-project/.git/hooks/commit-msg
       chmod +x joyjit-project/.git/hooks/commit-msg


(iii) Set the following git config value 
      Example:
       cd joyjit-repo
       git config jira.url "http://jira.mycompany.com"

(iv)  [Optional] If you wish jira integration to be triggered only
      on certain branches, add a comma-separated list of 
      branch names to git config "git-jira-hook.branches"
      For example:
   
        cd joyjit-repo
        git config git-jira-hook.branches "jira1,jira2"

      This will cause the integration to be triggered only on
      git branches jira1 and jira2. For instance, if you make a commit
      to branch "master", the hook will simply stay disabled.

      By default, if you do not set this config, all branches are 
      checked.

See the "Frequently Asked Questions" section to figure out what 
values to use for your "jira.url". 

3.2 Installation for "upstream" repository
------------------------------------------
(i)   Copy this script to 
      <upstream-repo-GIT-dir>/hooks/{commit-msg|post-commit|update|post-receive} 
      and mark it executable

      Example:
       cp git-jira-hook upstream-project.git/hooks/commit-msg
       cp git-jira-hook upstream-project.git/hooks/post-commit
       cp git-jira-hook upstream-project.git/hooks/update
       cp git-jira-hook upstream-project.git/hooks/post-receive
       chmod +x upstream-project.git/hooks/commit-msg
       chmod +x upstream-project.git/hooks/post-commit
       chmod +x upstream-project.git/hooks/update
       chmod +x upstream-project.git/hooks/post-receive

(iii) Set the following git config values (Note: gitweb.url config is 
      recommended, but Optional): "jira.url" "gitweb.url"
      Example:
      cd upstream.git
      git config jira.url "http://jira.mycompany.com"
      git config gitweb.url "http://git.mycompany.com/gitweb.cgi/p=upstream-project.git;a=commit;h="

(iv)  [Optional] If you wish jira integration to be triggered only
      on certain branches, add a comma-separated list of 
      branch names to git config "git-jira-hook.branches"
      For example:
   
        cd joyjit-repo
        git config git-jira-hook.branches "jira1,jira2"

      This will cause the integration to be triggered only on
      git branches jira1 and jira2. For instance, if you make a commit
      to branch "master", the hook will simply stay disabled.

      By default, if you do not set this config, all branches are 
      checked.
See the "Frequently Asked Questions" section to figure out what values 
to use for your "jira.url" and "gitweb.url"


4. Using git-jira-hook
======================
When you are read to make a git commit, make sure that you have an 
appropriate open jira issue. There can be more than one open issues.
Lets say this commit deals with Jira issues FOO-23 and BAR-42  and 
also marks FOO-56 as resolved.

Anywhere in your commit message, you must put the following strings 
(without the quotes):
   "refs #FOO-23"
   "refs #BAR-42"
   "fixes #FOO-56"



And then you do a "git commit" the normal way. At this time, assuming 
the "private" repository, the commit message will be checked for 
references to Jira issues and the commit will succeed only if these
issues exist.

And then, at a later time, when you do "git push" to push your changes
upstream, the final validation and Jira issue update will be done.

NOTE: The "fixes" feature does not work yet :<


5. Known Limitations
====================

I am working to fix all of these issues:
   * The "fixes" text does not yet mark the Jira issue as resolved.

   * The error messages  are a bit confusing (cluttered with too much detail).

DISCLAIMER: This is my very first attempt at writing python code, so it is not 
very well written.


6. Frequently asked Questions
=============================

Q 6.1  What value should I use for "jira.url" git config?
    A. It depends on your Jira server setup. When you log-in to the Jira 
       server using a browser, the URL to the login page typically looks like:
           http://jira.mycompany.com/secure/Dashboard.jspa 
       In which case, your "jira.url" should be "http://jira.mycompany.com"
 

Q 6.2  What value should I use for "gitweb.url" git config?
    A. Assuming you have gitweb enabled for your repository, this is the URL which you
       use to access gitweb.
       for instance, in order to view commit "424daa955f5c8a17aab9d524071f65f1999769a9"
       in gitweb, if you use:
http://git.mycompany.com/gitweb.cgi?p=joyjit-repo/.git;a=commit;h=424daa955f5c8a17aab9d524071f65f1999769a9

       Then the "gitweb.url" to use is:
       "http://git.mycompany.com/gitweb.cgi?p=joyjit-repo/.git;a=commit;h="


7. Credits
==========
This script was inspired by the following:
   http://github.com/dreiss/git-jira-attacher/tree/master
   http://confluence.atlassian.com/display/JIRAEXT/Jira+CLI


8. References
=============
[1] Git, an source configuratiin management ("SCM") tool
    http://git-scm.com/

[2] Jira,  a bug tracking system
    http://www.atlassian.com/software/jira

[3] Gitweb, a web based browser for git
    http://git.or.cz/gitwiki/Gitweb
