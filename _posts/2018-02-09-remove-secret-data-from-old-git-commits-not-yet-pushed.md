---
layout: post
title: Remove Secret Data from Old git Commits not yet pushed
---
I didn't push to the remote yet. The very initial commit of my repo has an API token in it.  
My current verison still has that API token in it.  
I want to replace the token at the initial commit, then I want to rebase all the other commits after that.  
To do this, I'll make a dummy branch at master so I don't lose that branch.  
`git branch dummybranch`  
Then I'll `git checkout init_commit_sha1`  
Then make a branch to name it, `git branch initcommit`.  
Then I will modify code here, removing the secret data.  
Oops!  I'm stupid.  I need to remove the initial commit from the history.  
  
So, now my plan is to create a new repository, starting with the working copy of the initial commit, but with no commits in this new repository yet.  
Then I will add my bad repo as a remote, fetch all the commits, and 
hopefully see a way to rebase the commits.  
  
Crap !  Turns out I have commits in the middle of the history that
has another occurrence of the secret data, then a commit where gofmt
indented the line with the secret data.  
You know what, I'll go with the Internet's advice.  I'll squash all the commits
into 1 commit, totally wasting all my history but whatever.  
What I'll do is add a new commit after master that removes all occurrences
of the secret data.  Then I'll squash everything, so the secret data is not in that squashed commit.  
  
Crap !  The squashed commit contains a deletion of an occurrence of the secret data!  
Well dang, I don't know what to do.  I'll just give up, start a new repo history from where I'm at now, after removing all occurrences of the secret data. 
 
