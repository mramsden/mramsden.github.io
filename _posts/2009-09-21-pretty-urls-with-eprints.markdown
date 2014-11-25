---
layout: post
title: Pretty URLs with EPrints
date: 2009-09-21 23:44:31.000000000 +01:00
categories:
- eprints
- perl
tags:
- apache
- eprints
- perl
status: publish
type: post
published: true
meta:
  _edit_last: '4005008'
author: Marcus Ramsden
---
Recently I have been working on a project which was focused on adding Homepages and Profiles for users to EPrints repositories called MicroViews. Currently when a user logs into an EPrints repository all you are presented with is your Manage Deposits page. With MicroViews the user will be presented with a Homepage which provides them with various pieces of information about the repository which is relevant to them, for example what are their most popular publications and if there are any issues with their publications that have been raised at the last audit.

The extension also adds a profile page to EPrints for each user. The idea of profile pages is to provide a static external facing page which the rest of the world can see. This page would include basic information about the user and the publications they have added to the repository. This effectively provides a biography of the user with regards to the repository.

One thing that we wanted to do is create pretty URLs for the user profiles of the form http://examplerepository.com/profile/. Initially we did this by making a modification to perl_lib/EPrints/Apache/Rewrite.pm but this was not the ideal situation for a plugin since this is a modification to the core code of EPrints. The problem with this approach is that if the repository administrator went on to update their core EPrints install then they would end up losing the hand made modification which is required for the plugin and they would have to add it again.

Instead we investigated creating a custom URL handler and adding it to the request handling chain using archives/ARCHIVEID/cfg/apachevhosts.conf. By doing this we won't change any core EPrints code and all of the changes will be safe from any auto generated changes. The first thing we did was create a custom rewrite handler which took the following shape;

{% highlight perl linenos %}
package EPrints::Plugin::MePrints::Apache::Rewrite;

use strict;
use warnings;
our @ISA = qw/ EPrints::Plugin /;
use EPrints::Apache::AnApache;

sub handler
{
  my( $r ) = @_;
  my $uri = $r->uri;
  my $repository_id = $r->dir_config( 'EPrints_ArchiveID' );
  if( !defined $repository_id )
  {
    return DECLINED;
  }
  my $repository = EPrints::Repository->new( $repository_id );
  if( $uri =~ m! ^\/profile\/([0-9a-zA-Z]+)$ !x )
  {
    # Find the user and if found set the appropriate filename for the response object.
    # If the user is not found then don't do anything or return anything and this will
    # cause EPrints to respond with a 404 error.
  }
  else
  {
    return DECLINED;
  }
}
{% endhighlight %}

Next we needed to add only one line to the archves/ARCHIVEID/cfg/apachevhost.conf file;

{% highlight perl linenos %}
PerlTransHandler EPrints::Plugin::MePrints::Apache::Rewrite
{% endhighlight %}

After a server reset we had the pretty URLs for the profile pages. This works thanks to the PerlTransHandler statement in the apachevhost.conf file. What this does is add a handler to the chain of handlers to be run. By default in EPrints there is only one handler and that is the default one. What we have done with the change above is add a handler into the chain before that. The handler method in EPrints::Plugin::MePrints::Apache::Rewrite gets called and if it gets a match for profile then it goes ahead and handles it. The "return DECLINED" tells Apache that it should move onto the next URI translation handler in the chain.
