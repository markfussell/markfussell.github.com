---
layout: post
title: "Being a git about everything (Annexing)"
date: 2013-01-13 18:16
comments: true
categories: git
---

This is the second in a series of using git as part of interesting solutions to problems.

The first is here: [Intro](/blog/2013/01/11/git-about-everything-intro/)

## Dealing with Binary Files

As mentioned in the first posting, git and similar DVCS have issues with binary files.  Adding 100s of 10MB files, or 100s of
versions of 10MB files will produce gigabytes worth of data that must be cloned by everyone using the repository.  How do we avoid this?

There are a number of solutions in this space out there with differing characteristics, but the core approach is usually a similar
"Don't store it in git".  Instead we want to record enough information to retrieve the binary files from somewhere else.

<!-- more -->

## Record Enough of the Binary File

What is
enough information?  How about 160 bits!  Using SHA1 or any similar hash, we can identify the contents of any file.  To add
a bit of consistency and readability, we will make this hex-based, so we get 40 characters.  And to make it a little clearer what
hash was used and what this string is, we add 'sha1_' to the front and '.blob' to the end.  Now our 10MB file has become 50 to 52 bytes depending on
human-entered white spaces.  For example:

   * sha1_8ac02ee34b94461fed19320d789f251e6a2a6796.blob
   * SHA1-Hash = 8ac02ee34b94461fed19320d789f251e6a2a6796
   * Google test: [http://www.google.com/search?q=8ac02ee34b94461fed19320d789f251e6a2a6796](http://www.google.com/search?q=8ac02ee34b94461fed19320d789f251e6a2a6796)

And we have confirmed that the hash is enough to identify that file's content (and the file itself if it has a unique name).

Storing tens of thousands of 50Byte files is still under a few megabytes, so that part is good.

## Put the content into the Cloud (or similar)

Where should we store the actual content?  Pretty much anywhere we want if things are simple (only a few files at a time to store and retrieve, need
only local access), but
if we want to store and retrieve thousands of files from anywhere rapidly things get nastier.  For example, there are projects that try to
hide the process of file-contents being moved elsewhere using git smudge/clean filters.  Unfortunately this make the process all of: sequential, heavy, and
perpetual.  We would have similar major issues if we moved the content via some other sequential approach that was not super-fast.  And
git is meant to be highly distributed, so our approach needs to work for all the 'clones' of a repository.  And finally, we don't
want to break our bank over this.

The solution?  Amazon S3 or similar.  Amazon S3 is:

   * Relatively inexpensive for what it does (about a dime a month for 1GB)
   * Highly distributed
   * Reasonably fast in network transfers
   * Incredibly fast as you throw more workers at the network transfers
   * Blazingly fast if you throw many workers from EC2 at it

So if we want to move files from our Git repository into S3, how do we do that?  This isn't really a git issue at all if we do
it before 'add' and after 'pull'.  It becomes
a simple filesystem-to-S3 issue and s3cmd [http://s3tools.org/s3cmd](http://s3tools.org/s3cmd) is a very well trusted tool for this.
Add in the parallel patch [https://github.com/pcorliss/s3cmd-modification](https://github.com/pcorliss/s3cmd-modification) and
you can have any numbers of workers running.  We want to be a 'git' about everything, but we can solve this issue independently
of git and combine the two.  The only large remaining issues are the actual processes of:

   * 'deflating': moving the content of a file somewhere and leaving a content-hash in its place.
   * 'inflating': replacing a content-hash with the actual content

A highly annex-augmented version of s3cmd is here [https://github.com/markfussell/s3cmd-modification](https://github.com/markfussell/s3cmd-modification)
and was done while working at [Rumble](http://www.rumblegames.com) where we needed to move gigabytes of high-quality art assets around
as part of the build-deploy pipeline.

## Working with S3 Annexed Git Repositories







## Alternatives

There are some alternative approaches out there, which I should mention.  Some I tried and they didn't perform well
enough.  Others didn't match the needs we had at Rumble or my needs outside of Rumble.  But they are interesting
projects:

   * [http://git-annex.branchable.com/](http://git-annex.branchable.com/)
   * [https://github.com/schacon/git-media](https://github.com/schacon/git-media)






