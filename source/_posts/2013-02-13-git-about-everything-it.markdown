---
layout: post
title: "Being a git about everything (IT)"
date: 2013-02-13 18:16
comments: true
categories: git
---

This is the third in a series of using git as part of interesting solutions to problems.

The first is here: [Intro](/blog/git-about-everything-intro/)

## Leveraging git to help with IT tasks

Doing IT for computers involves installing software, configuring things, doing backups, updates, etc.
Git can help out with a lot of this, with several big benefits.  Some examples:

   * IT changes should always be version controlled in case of mistake.  Git and a couple patterns/tools makes that easy
   * Installing software can frequently be done with 'yum' and similar commands, but some software needs a package, and some times you want more direct 'this version' control.  An annexed git repository helps with that
   * Doing backups or other kinds of off-lining with an annexed repo is very simple and flexible
   * Git enables a PushMePullYou model that is very flexible and capable of driving hundreds of machines to do either (or both) very light and very heavy-weight operation

Some of these things can be done with other tools, but I have found an annexed git repository to make doing these things easier and better

<!-- more -->

## IT Changes

Any time you have a running computer you should be careful about making changes to it that you can't
easily revert.  The classic `rm -fr /` is amongst the most painful, but even a few simple tweaks of a
configuration file could cause you minutes or hours of pain if you do them wrong and can't just
start over.

There are many solutions to this problem:

   * Create a '.bak' file
   * Create a '.datestamp' file of the file before you touch it
   * Copy the file into a backup folder
   * Rsync the directory somewhere else
   * Version control the whole directory structure
   * Have frequent backups of the whole machine

Git can help with a number of different approaches, but
I will start with one approach that I find to be among the simplest, most-general, and most-beneficial.

### Copy 'changing' files into an IT repo

To get a huge step up on the problem, we are going to allocate an annexed git repository to 'being for IT'.
And further we will have every computer out there have its own directory within the repository.  A computer's
directory is simply it's name (e.g. `hostname` in unix systems).  This give us a structure like this:

   * IT-Repository (e.g. giteveryrepo2)
      * it/comp/
         * {computer1}
            * Any files you want with their full path as if under '/'
         * {computer2}
         * {computer3}


As long as computers have unique names, we can freely work with any computer's files without collision
with another computer.  And because Git is distributed, we can work on files _on_ any computer and
merging is very unlikely to be an issue.


An example of an IT repository is at:

   * [https://github.com/markfussell/giteveryrepo2](https://github.com/markfussell/giteveryrepo2)

and more specifically, the directory for computers is at:

   * <https://github.com/markfussell/giteveryrepo2/tree/master/it/comp>

As of this writing, there are two computers listed.  Both are EC2 machines, so they have the standard
patterns for AWS machine naming.

   * domu-12-31-39-0e-e2-53
   * ip-10-28-81-39

On each machine, I ran a script to copy from system files to the repository.  The scripts are in repo2/bin/copy
so a sequence looks like this

```bash
./bin/copy/copyFromSystemFile.sh /var/www/html/index.html
git pull; git add it; git commit -m "Server2"; git push
```

Again, because the machines have their own directories, merging should be automatic no matter what order
or how much time-overlap occurs.  Two notes:

   * The machine names are lower-cased to avoid confusing behavior.  I strongly recommend keeping directories all lowercase
   * If you have any binary files you are copying from the machine, you need to remember to 'deflateAll' the repository before the git line.



#### Flattening Computers next to each other

An advantage of flattening out computers next to each other is you can compare them:

```bash
diff -r it/comp/*
diff -r it/comp/domu-12-31-39-0e-e2-53/var/www/html/index.html it/comp/ip-10-28-81-39/var/www/html/index.html
1c1
< domU-12-31-39-0E-E2-53
---
> ip-10-28-81-39
```

In this case it should be apparent that the files have the `hostname` inside them, so that is what is changing.

So you can compare the history of a particular computer by looking through git history, and you can compare two
different computers to each other by comparing the different directories that each computer owns.

### Copy files from the IT repo

There is also a script to copy from the IT repo to a particular system location.  This is a little more
risky, but can be useful at times.  Effectively this would roll back your local change if you didn't touch
the repo from anywhere else, and it can be used to document a proposed change before applying it.  This
leads to more of 'git for IT automation' though, which I will cover later.

## Software installation

Software installation is the major part of machine provisioning.  Unless you have a fully-baked AMI or
really are happy with the factory install, you need to add more software to make your computer useful.
Installing software can frequently be done with 'yum' and similar commands, but some software needs a
package install (e.g. with 'rpm') and some times you want more direct control of the version installed.

Fetching RPMs from the web at install time is a major source of automation failure and annoyance.  Sites
go down and if you are fetching from many different sites, you could have a computer with 9 out of 10
things installed on it.  Generally I try to make machine provisioning _all or nothing_ : Either everything
gets installed properly or I just wipe the machine and try again.  Primordial scripted installs are
easy to write (no human interaction), but trying to recover from a partial install is
a way to become very unproductive (potentially very angry too).  Additionally, some sites can be slow
and you don't want to be penalized by that for every computer you provision.

A solution is to store RPMs and the like in an Annexed Git repository.  S3 is both very fast and very reliable,
with an SLA that 'guarantees' four nines (99.99%) of uptime a year, which is < 1-hour of downtime.  If
all install packages are stored in S3, provisioning would be almost solely dependent on S3, so it would
always be an _all (99.99%) or nothing (00.01%)_ provisioning model.

### Example and idioms

You could put RPMs and the like anywhere you want since the annexing is orthogonal to file location,
but I conventionally put things in 'it/resource'

   * <https://github.com/markfussell/giteveryrepo2/tree/master/it/resource>

An annexed repo will also tend to have a '.temp' directory in the '.gitignore', so that is a handy place
to do temporary work without risk of colliding with changes to the repository itself.  When you inflate a file,
it is 'dirty' / changed so it could conflict with a subsequent merge and you would have to either deflate
or 'git checkout' the file/s to clean up.  Worse would be an archive that expanded into a directory structure,
so it is generally nice to work in '.temp' where these aspects don't cause issues.


In giteveryrepo2 there is a Java-7 rpm, which can be installed as follows:

```bash
pushd /root/gitrepo/giteveryrepo2
git pull
mkdir -p .temp
cp it/resource/jdk-7u13-linux-x64.rpm .temp
./bin/inflatePaths.sh .temp/
rpm -i .temp/jdk-7u13-linux-x64.rpm
/usr/java/default/bin/java -version
```

Similarly for installing Apache Tomcat:

```bash
pushd /root/gitrepo/giteveryrepo2
git pull
mkdir -p .temp

cp it/resource/apache-tomcat-7.0.35.tar.gz .temp
./bin/inflatePaths.sh .temp/

pushd .temp
tar -xvf apache-tomcat-7.0.35.tar.gz
mv apache-tomcat-7.0.35 /usr/share
ln -s /usr/share/apache-tomcat-7.0.35 /usr/share/tomcat7


popd
```

And just to show the install actually works.

```bash
cat > /etc/httpd/conf.d/proxy_ajp.conf <<EOS
ProxyPass /tomcat/ ajp://localhost:8009/
EOS

service httpd restart

/usr/share/tomcat7/bin/startup.sh

```

should bring up a standard Tomcat welcome screen.

<img width="743" height="352" src="/images/git-about-everything-it/TomcatScreenshot1.png" />


## Backups

Backing up database dumps, logs, etc. is as easy as copying them into an appropriate folder within the repo and deflating.
This is similar to IT changes: you just put things in a reasonable location on the machine and then

```bash
./bin/copy/copyFromSystemFile.sh {reasonable-location}
./deflateAll.sh
git pull; git add it; git commit -m "Backup of `hostname` as of `date`"; git push
```

This bascially works fine, with two minor issues:

   * You probably want the backup repo to be different from the IT repo even though their layout would be similar.  You wouldn't want a bunch of 'backup' activity happening on something you are using for IT maintenance or even machine provisioning.  The rhythm and types of commits would be inconsistent.
   * At some point, you are going to have _too many backups_ and you need to get rid of some noise.  This is 'cleaning the annex'


### Cleaning the Annex

By default the Annex has a copy of every 'big' file you have ever annexed in the history of the repository.  Given S3 doesn't
cost very much, this may never be an issue for you.  But if it is an issue, you need to throw out the garbage.  There
are fancier versions of this process but the simplest involves:

   * Removing any files you no longer want to keep around from the current HEAD of git.  So if you have 1000 backups in the filesystem and want less than that, delete and commit the ones to get rid of.  Ideally this is completely automated as part of normal activities (e.g. new backups cause some older backups to be deleted)
   * Inflating the whole repository from the current Annex (Annex2a)
   * Changing the current Annex to point to a new empty Annex (Annex2b)
   * Deflating the whole repository to the new Annex (Annex2b)
   * Committing the change of Annex into git

Note that at this point you have the option to delete the old annex or not.  If you leave it around you have a 'shiny new'
cleaner Annex2b which can make some things go faster, but if you ever want to go back to an old version it will work too.
Because the annex information is in the version, checking out an old version will automatically switch the annex to use
with it too.

The 'moveToNewAnnex.sh' script looks like this:

```bash
die () {
    echo >&2 "$@"
    exit 1
}

[ "$#" -eq 1 ] || die "1 argument required, $# provided"
echo $1 | grep -E -q '^s3://' || die "S3 bucket URL required, $1 provided"

export NEW_ANNEX="$1"

./bin/inflatePaths.sh .

echo $NEW_ANNEX > s3info/s3annex_config.txt

./deflateAll.sh
```

An example using the contents of 'giteveryrepo2':

```bash
#Remove the temp directory since we don't want to annex it
rm -fr .temp

./bin/moveToNewAnnex.sh s3://emenar.com/gitevery/giteveryrepo2v2/
git status
git diff s3info/s3annex_config.txt

#Go back to previous annex again

git checkout -- s3info/s3annex_config.txt
git status
```

It would be 'more optimal' to go directly from S3 -> S3, but the above works well enough if you have decent bandwidth
and especially if you are within EC2.

## Summary

Annexed git repositories can make certain IT activities much easier, more reliable, or more powerful (diff two machines' configurations).
The above wasn't showing IT automation, but it shows a valuable tool for IT automation as well as more manual interactions.

## Next

Our next problem will be...





