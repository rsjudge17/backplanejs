# Introduction #

This page will describe how to submit a patch to backplanejs. However, for the time being I'm using it to keep my notes on how to incorporate changes from [Anton](http://code.google.com/r/creavenmoro-backplanejs/) and [Frankie](http://code.google.com/r/fdintino-backplanejs/).

## Environment ##

Each developer wishing to contribute patches should have their own clone of the backplanejs repo. This can be with any Mercurial provider, such as Google Code or Bitbucket. Please set the license to Apache.

In addition to installing Mercurial you will also need the [pbranch extension](http://bitbucket.org/parren/hg-pbranch).

## Branches ##

The default branch should never be used for code changes, since it is used to keep the cloned repo in sync with the master repo at backplanejs.

All fixes and new features can be created as branches using `pbranch`. To keep synchronised, periodically `pull` from the master repo and use `pmerge` to bring any changes into the patch branches.

Any patch branch that is not dependent on another should be dependent on the default branch. This allows patches to be applied separately.

## Submitting a patch ##

Details to follow.

## Incorporating the patch into backplanejs ##

  1. create a local copy of the developer's clone;
  1. optionally, create an alias for the repo (e.g., `frankie` or `anton`);
  1. do a pull from the remote clone into the local copy of the clone (`hg pull -R anton`);
  1. import the revision required into the backplanejs patch queue (`hg export -R anton b30b992ee4 | hg qimport -n issue-92 -`).

To import a patch branch:
```
hg pull -b patch-ft-insert-at frankie
hg pgraph --tips --as-text > .hg/pgraph
```