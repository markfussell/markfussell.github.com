---
layout: post
title: "Being a git about everything (IT Automation [2])"
date: 2013-02-19 18:16
comments: true
categories: Git IT Automation Cloud
---

This is the fifth in a series of using git as part of interesting solutions to problems.

The first is here: [Intro](/blog/git-about-everything-intro/) and the previous
part of this topic is here: [IT Automation](/blog/git-about-everything-it-automation/)

## PushMePullYou or Leveraging git to enable mass-automated IT

<div style="float:right">
<img width="244" height="191" src="/images/git-about-everything-it-automation/Pushmepullyou_mlf1c.png" />
</div>

The previous post dealt with the groundwork of having Git be a central part of IT automation.  That
showed the core idea but was a bit too simple to fully express the power of the approach.
This post will be dealing with all the things that were left off, especially support for:

   * Many different types of servers with both their own and shared 'recipes'
   * More complicated install/upgrade actions
   * More sophisticated install behavior
   * Multiple versions of 'recipes' and an ability to promote whole IT from development to production
   * Getting information from other active repositories

<!-- more -->

To just jump-in and not be so incremental, I want to build the following deployment:

   * A load-balancing layer that registers itself with the outside world and knows how to talk to the application layer
   * An application layer that runs an application which may be updated at any time
   * A database layer using a cluster of Riak servers
   * A presence server that can record what servers are present (so other servers can leverage)

Each of these will be a different stack for easier management.  There will also be a Control server
which makes setting up the deployment easier (for example, it will make sure you have the latest AWS CLI).

## The Control Server

To get everything up and running check out the repository:

```bash
git clone git://github.com/markfussell/giteveryrepo4.git
```

And then in the Amazon AWS console launch the control server with the file template at `it/aws/cloudformation/GitEvery4ControlServer.template`.

<img src="/images/git-about-everything-it-automation-2/create_control_stack.png" />

You can then SSH into the server and `sudo su -` to change to root. And `cd gitrepo/giteveryrepo4` to get into the root of the repository.

<img src="/images/git-about-everything-it-automation-2/control_server_login.png" />

## Handling Different Types of Servers

A major difference from the previous example is there are now several types of servers, and they
will have different:

   * Firewall permissions
   * Initial setup of software
   * Ongoing configuration changes

There are also a number of similarities and ideally the CloudFormation files are as similar and
simple as possible.

### Firewall Permissions

One major change is to get rid of stack-generated security groups: these are difficult to manage since
they have ever-changing and obscure names.  I believe it is better for any real deployment to control
the security groups independently of the Stacks.  So now we have two kinds of security groups:

   * One for the `deployment` as a whole
   * One for each `part` a server node can be

The `deployment` and `part` are assigned as constants in the Stack template mappings:

```json
  "Mappings" : {
    "TemplateConstant" : {
       "stacktype" : { "value" : "GitEvery4LbServer" },
       "initgitrepo" : { "value" : "giteveryrepo4" },
       "nodepart" : { "value" : "applbnode" },
       "deployment" : { "value" : "testdeployment" }
    },
```

and then later referenced in the security group section:

```json
        "SecurityGroups" : [ { "Fn::FindInMap" : [ "TemplateConstant", "deployment", "value" ] }, { "Fn::FindInMap" : [ "TemplateConstant", "nodepart", "value" ] } ],
```

Before we create the stack, we create the appropriate groups.  For example, the `deployment` group will enable SSH into
the nodes and any node within the deployment to talk to any other node:

```bash

#===================
#=== testdeployment
#===================

ec2-create-group testdeployment -d  testdeployment
export OWNER=`ec2-describe-group | grep GROUP | head -n 1 | cut -f 3`

#Enable SSH In
ec2-authorize testdeployment -p 22 -s 0.0.0.0/0

#Enable deployment to talk to itself
ec2-authorize testdeployment -o testdeployment -u ${OWNER}

```

### Initial setup of software

Within the CloudFormation template, we use the same approach as last time: simply checkout a git repository and call into it.  In Bash it would be:

```bash
yum -y install git
mkdir /root/gitrepo
cd /root/gitrepo
git clone git://github.com/markfussell/`cat /root/nodeinfo/initgitrepo.txt`.git

cd /root/gitrepo/`cat /root/nodeinfo/initgitrepo.txt`
include () { if [[ -f "$1" ]]; then source "$1"; else echo "Skipped missing: $1"; fi }
include bin/init/common/init.sh
```

which is converted to JSON (string-quoted) within the template.

This entering on a common `init.sh` entrypoint makes the CloudFormation stacks simpler and more general.  It is much
easier to update and push to the git repository than to update all the stacks that are using the repository.

The new part is now at the end of the common entrypoint: jumping into more specific initialization depending
on the properties of the node:

```bash
include bin/init/part/`cat /root/nodeinfo/nodepart.txt`/init.sh
include bin/init/stacktype/`cat /root/nodeinfo/stacktype.txt`/init.sh
include bin/init/stacktype/`cat /root/nodeinfo/stacktype.txt`/part/`cat /root/nodeinfo/nodepart.txt`/init.sh
```

We so far have three variations:

   * The `part`
   * The `stacktype`
   * Combining both of the above.

We can be as general or as specific as we want.

A `stacktype` is simply the name of the template (vs. the name of an instantiated stack which has to be unique).
The main advantage of `stacktype` is it allows easy separation of behavior by kind of stack and
also allows versioning if `stacktype` includes a version number.

A `part` is the singular role a node plays within a deployment.  I use `part` instead of `role` to avoid
conflict with Chef where a node can have many roles.  A node has exactly one `part` it plays in the deployment,
and each `part` can have any number of `roles`.  A `part` should normally be fairly universal.  In our case
the five parts are `applbnode` (The main application load balancer), `appnode` (The main app), `riaknode` (The
riak database node), `presencenode` (A server presence recording server), and `controlnode` (The main launching
control server, which isn't really part of the running deployment).

With just the application and load-balancing parts/nodes running along with the controlnode, we have something like this:

<img src="/images/git-about-everything-it-automation-2/pushmepullyou_lbandappawsarch.png" />

Each `part` has its own stack template and instance, which creates one or more nodes of that `part` type.



...More To Come...


## References

   * <http://bitfieldconsulting.com/scaling-puppet-with-distributed-version-control>
   * <http://blog.afistfulofservers.net/post/2012/12/21/promises-lies-and-dryrun-mode/>


## Next

Our next problem will be...





