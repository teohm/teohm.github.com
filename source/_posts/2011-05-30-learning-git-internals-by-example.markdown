---
layout: post
title: Learning Git Internals by Example
description: >
  The post explains basic Git internals (what happens in .git directory
  when you run a git command) using examples and pictures.  
comments: true
image: /images/tags/git.png
keywords: git, version control, git internals, visual example, graphical diagram.
categories: 
- version-control
- git
---

*Status: Draft.*  
*Plan to revise this post, probably simplify it in future..* 

Movitation
---
After switching to Git from Subversion and Mercurial for a few months, somehow I feel that Git is fundamentally different from Subversion or Mercurial, but couldn't really tell the differences. I often see terms like tree, parent etc. in GitHub, which I have no idea what they actually mean.

So I decided to spent some time to learn Git. 

I will try to summarize and publish important stuffs I learned about Git along the way.. but here is the first entry, about Git internals, which helped me to answer how Git is different other source control tools.


Objects, References, The Index
---
To understand the core of Git internals, there are 3 things to we should know: **objects, references, the index**. 

I find this model is elegant. It fits well in a small diagram, as well as in my head. 

![A picture illustrates files in .git directory mentioned in this article.](/images/2011-05-30-learning-git-internals-by-example/big-picture.png)

### Objects
All files that you commited into a Git repository, including the commit info are stored as **objects** in `.git/objects/`.

An object is identified by a 40-character-long string -- SHA1 hash of the object's content.

There are **4 types** of objects: 

1. *blob* - stores file content.
2. *tree* - stores direcotry layouts and filenames.
3. *commit* - stores commit info and forms the Git commit graph.
4. *tag* - stores annotated tag. 

The example will illustrate how these objects relate to each others. 

### References
A *branch*, *remote branch* or a *tag* (also called *lightweight tag*) in Git, is just a **pointer to an object**, usually a commit object. 

They are stored as plain text files in `.git/refs/`.

#### Symbolic References
Git has a special kind of reference, called *symbolic reference*. It doesn't point to an object directly. Instead, it **points to another reference**.

For instance, `.git/HEAD` is a symbolic reference. It points to the current branch you are working on. 

### The Index
The index is a staging area, stored as a binary file in `.git/index`. 

When `git add` a file, Git adds the file info to the index. When `git commit`, Git only commits what's listed in the index.

---------

Examples
---
Let's walkthrough a simple example, to create a Git repository, commit some files and see what happened behind the scene in `.git` directory.

### Initialize New Repository
{% highlight bash %}
$ git init canai
{% endhighlight %}

![illustration](/images/2011-05-30-learning-git-internals-by-example/init.png)

What happened:

- Empty `.git/objects/` and `.git/refs/` created.  
- No index file yet.
- `HEAD` symbolic reference created.
      $ cat .git/HEAD 
      ref: refs/heads/master


### Add New File
{% highlight bash %}
$ echo "A roti canai project." >> README
$ git add README
{% endhighlight %}

![illustration](/images/2011-05-30-learning-git-internals-by-example/new-file.png)

What happened:

- Index file created.  
  It has a SHA1 hash that points to a blob object.
      $ git ls-files --stage
      100644 5f89c6f016cad2d419e865df380595e39b1256db 0 README
- Blob object created.  
  The content of README file is stored in this blob.
      # .git/objects/5f/89c6f016cad2d419e865df380595e39b1256db
      $ git cat-file blob 5f89c6
      A roti canai project.


### First Commit
{% highlight bash %}
$ git commit -m'first commit'
[master (root-commit) d9976cf] first commit
 1 files changed, 1 insertions(+), 0 deletions(-)
 create mode 100644 README
{% endhighlight %}

![illustration](/images/2011-05-30-learning-git-internals-by-example/first-commit.png)

What happened:

- Branch 'master' reference created.  
  It points to the lastest commit object in 'master' branch.
      $ cat .git/refs/heads/master 
      d9976cfe0430557885d162927dd70186d0f521e8
- First commit object created.  
  It points to the root tree object.
      # .git/objects/d9/976cfe0430557885d162927dd70186d0f521e8
      $ git cat-file commit d9976cf
      tree 0ff699bbafc5d17d0637bf058c924ab405b5dcfe
      author Huiming Teo <huiming@favoritemedium.com> 1306739524 +0800
      committer Huiming Teo <huiming@favoritemedium.com> 1306739524 +0800

      first commit
- Tree object created.  
  This tree represents the 'canai' directory.
      # .git/objects/0f/f699bbafc5d17d0637bf058c924ab405b5dcfe
      $ git ls-tree 0ff699
      100644 blob 5f89c6f016cad2d419e865df380595e39b1256db  README


### Add Modified File
{% highlight bash %}
$ echo "Welcome everyone." >> README
$ git add README 
{% endhighlight %}

![illustration](/images/2011-05-30-learning-git-internals-by-example/modified-file.png)

What happened:

- Index file updated.  
  Notice it points to a new blob?
      $ git ls-files --stage
      100644 1192db4c15e019da7fc053225d09dea14bc3ac07 0 README
- Blob object created.  
  The entire README content is stored as a new blob.
      # .git/objects/11/92db4c15e019da7fc053225d09dea14bc3ac07
      $ git cat-file blob 1192db
      A roti canai project.
      Welcome everyone.

### Add File into Subdirectory
{% highlight bash %}
$ mkdir doc
$ echo "[[TBD]] manual toc" >> doc/manual.txt
$ git add doc
{% endhighlight %}

![illustration](/images/2011-05-30-learning-git-internals-by-example/subdir.png)

What happened:

- Index file updated.
      $ git ls-files --stage
      100644 1192db4c15e019da7fc053225d09dea14bc3ac07 0 README
      100644 ea283e4fb22719fad512405d41dffa050cd16f9a 0 doc/manual.txt
- Blob object created.
      # .git/objects/ea/283e4fb22719fad512405d41dffa050cd16f9a
      $ git cat-file blob ea283
      [[TBD]] manual toc


### Second Commit
{% highlight bash %}
$ git commit -m'second commit'
[master 556eaf3] second commit
 2 files changed, 2 insertions(+), 0 deletions(-)
 create mode 100644 doc/manual.txt
{% endhighlight %}

![illustration](/images/2011-05-30-learning-git-internals-by-example/second-commit.png)

What happened:

- Branch 'master' reference updated.  
  It points to a lastest commit in this branch.
      $ cat .git/refs/heads/master 
      556eaf374886d4c07a1906b9fdcaba195292b96
- Second commit object created. 
  Notice its 'parent' points to the first commit object. This forms a commit graph.
      $ git cat-file commit 556e
      tree 7729a8b15b747bce541a9752a8f10d57daf221b6
      parent d9976cfe0430557885d162927dd70186d0f521e8
      author Huiming Teo <huiming@favoritemedium.com> 1306743598 +0800
      committer Huiming Teo <huiming@favoritemedium.com> 1306743598 +0800

      second commit
- New root tree object created.
      $ git ls-tree 7729
      100644 blob 1192db4c15e019da7fc053225d09dea14bc3ac07  README
      040000 tree 6ff17d485bf857514f299f0bde0e2a5c932bd055  doc
- New subdir tree object created.
      $ git ls-tree 6ff1
      100644 blob ea283e4fb22719fad512405d41dffa050cd16f9a  manual.txt


### Add Annotated Tag
{% highlight bash %}
$ git tag -a -m'this is annotated tag' v0.1 d9976
{% endhighlight %}

![illustration](/images/2011-05-30-learning-git-internals-by-example/annotated-tag.png)

What happened:

- Tag reference created.  
  It points to a tag object.
      $ cat .git/refs/tags/v0.1 
      c758f4820f02acf20bb3f6d7f6098f25ee6ed730
- Tag object created.
      $ git cat-file tag c758
      object d9976cfe0430557885d162927dd70186d0f521e8
      type commit
      tag v0.1
      tagger Huiming Teo <huiming@favoritemedium.com> 1306744918 +0800

      this is annotated tag


### Add new (lightweight) tag
{% highlight bash %}
$ git tag root-commit d9976
{% endhighlight %}

![illustration](/images/2011-05-30-learning-git-internals-by-example/new-tag.png)

What happened:

- Tag reference created.  
  It points to a commit object.
      $ cat .git/refs/tags/root-commit 
      d9976cfe0430557885d162927dd70186d0f521e8

## More Readings
- "Chapter 7: Internals and Plumbing" in [The Git Community Book](http://book.git-scm.com/index.html).
- "Chapter 9: Git Internals" in [Pro Git](http://progit.org/book/ch9-0.html).

## What's Next?
Looking for a minimal git workflow suitable for a distributed team, long-running project.. 


