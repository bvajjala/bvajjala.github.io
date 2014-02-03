---
layout: post
title: "Moving from a self hosted Wordpress blog to Blogger"
date: 2012-10-21 15:15
comments: true
categories: [ blogger, perl, script, wordpress ]
---

I have a self hosted [wordpress food blog](http://xesla.ro) hosted on an old silly domain that I've wanted to move away from for a while.   I also want to stop paying hosting fees some time soon.   Since I'm already looking at moving a lot of my day-to-day activities to the 'cloud'  it made sense to pick a blogging platform that ties into a major cloud hub.   Google's Blogger was the obvious choice as I'm already using the Google Apps platform for my new domain `paulcz.net`.  

Creating a blog on blogger is dead simple.   I went ahead and created two: [food.paulcz.net](http://food.paulcz.net) for the new food blog and [tech.paulcz.net](http://tech.paulcz.net) to start journalling random tech things.     Transfering the blog content itself  is quite simple,  however, doing it in a way as to preserve links between posts, from other sites, and teaching the search engines how to find your new site requires a little more trickery.  

<!--more-->

I'm assuming you already have some technical know-how ...  this isn't for the Octogenarians or Luddites in the audience.     You'll need access to a linux host and have a basic understanding of editing files in vim and navigating the linux  command prompt as well as be able to navigate the Wordpress  and Blogger interfaces.

These examples  use my own blog so where you see [food.paulcz.net](http://food.paulcz.net) please modify the text to your own blog URL if you want to use them.   Some of the scripting is a bit silly and could be cleaner but they're throwaway scripts and I didn't want to spend too much time on them.

## Step 1.   Perform Initial Import/Export of blog.

Log into your wordpress blog and export the blog to an XMLfile ( `wordpress_export.xml` ).     You should be able to do this by going to `Dashboard -> Tools -> Export -> Download Export file`.

Browse  to [http://wordpress2blogger.appspot.com/](http://wordpress2blogger.appspot.com/).   This site will convert the file from Wordpress format to Blogger format.   Upload and Convert your file, saving it to `blogger_export.xml`.

Log into blogger and create a new blog.   if you're using your own domain set up the address in basic settings and then go to `Other Settings -> Import blog`  and select the `blogger-export.xml` file.   You'll also need to publish all the posts ( they don't seem to publish by default ).  You can do this  50 at a  time from the Posts section.

Now if you have a small blog, or don't have any intrasite links you're basically done here.   However A lot of my posts have links back to other posts and these are not converted.   This means that there's tons of links on my new blog posting back to my old blog.    

Thankfully I don't upload images to the blog, rather link to them on my picasa/google+ albums.   This means I don't have to deal with some wacky image stuff.

## Step 2.   Modify links to point to new site.

The first thing to do is get a list of all blog post links on your new Blogger site.   This is pretty easy,  you can do it via an RSS feed and some perl magic.

Browse to your new blog like so:   [http://food.paulcz.net/feeds/posts/default?start-index=1&max-results=999](http://food.paulcz.net/feeds/posts/default?start-index=1&max-results=999).   This  will give you a page showing your entire blog ( assuming you have less than 999 posts ).    Save this file from your browser as default.xhtml.

now create a perl script called `urls.pl` ...    notice the URL ( with \ escaped characters ) .. you'll need to modify this to match your new blogger url.

``` perl urls.pl 
#!/usr/bin/perl
open (TXT,"< default.xhtml");
while (<TXT>) {
 chomp;
 /(http:\/\/food\.paulcz\.net\/\d\d\d\d.*?html)/;
 print $1 . "\n";
}
```

Then run the following:

``` bash Terminal
perl urls.pl | uniq > blogger_urls.txt'
grep "<link>" wordpress_export.xml | sed "s/<link>//" | sed "s/<\/link>//" \
 | perl -e 'print reverse <>' > wordpress_posts.txt
cp blogger_export.xml blogger_munge.xml
vim -O wordpress_posts.txt blogger_urls.txt
```

This  will extract a unique list of URLs  of posts from both your new blogger and old wordpress sites. Chances are they won't match up exactly and some hand editing will be required.  That's okay the  last line  above will open the  files side-by-side in vim  to allow you  to clean this  up.     Hand edit each side  to ensure that both URLs on the same line match the same a posts.

Once that is done we can write  and run some more perl to create a shell script  that will modify the  `blogger_munge.xml` file ( copied from the original `blogger_export.xml` above ) replacing all your old links.

Create the following perl script named `create_sed.pl`

``` perl create_sed.pl
#!/usr/bin/perl
my @combined;
open(TXT, "< wordpress_posts.txt");
my @wordpress = <TXT>;
close TXT;
open(TXT, "< blogger_urls");
my @blogger = <TXT>;
chomp(@wordpress);
chomp(@blogger);
$size = $#blogger;
for ( $count=0;$count<=$size;$count++ ) {
 $string = $wordpress[$count] . "|" . $blogger[$count];
 push(@combined, $string);
}

foreach ( @combined ) {
 s/\//\\\//g;
 s/\./\\./g;
 s/\|/\//;
 print "sed -i \'s/$_/g\' blogger-munge.xml\n";
}
```

Now run `create_sed.pl`, and the resultant `munge.sh` script.

``` bash Terminal
perl create_sed.pl  > munge.sh
sh munge.sh
```

now you have a blogger_munge.xml file that links back to its own articles.    There may be some residual links  back to your old  site.   If you're a perfectionist you  could hand edit  the XML to fix this,  but I'd say its now good  enough.

## Step 3. Re-Create your blogger blog.

Now you'll want to delete your initial blogger blog and recreate it.   Remember to set your URL again, then import the `blogger_munge.xml` file and republish all the posts.

## Step 4.  Set redirects from your old blog space.

Since we have the matching pairs of URLs  it becomes quite simple to set up some permanent redirects from your apache config, or even better from inside a `.htaccess` file.  Setting a permanent redirect not only redirects the user to the correct place,  but in theory also informs a search engine ( on its next pass over your blog )  to update its database to the new location.  

Links from around the web pointing at your old domain will continue to work for as long as the redirects work.    At some point if you want to retire your old domain this will break those links...   But there's not a great deal you can do about that.  

save the following perl snippet as `rewrite.pl`.  remember to rewrite any strings specific to my domains to match your own: 

``` perl rewrite.pl
#!/usr/bin/perl
my @combined;
my @wordpress;
open(TXT, "< wordpress_posts.txt");
while (<TXT>) {
 chomp;
 s/\/$//;  # remove trailing /
 my @temp = split "/";
 $last = pop @temp;
 push(@wordpress, $last);
}
close TXT;
open(TXT, "< blogger_urls");
my @blogger = <TXT>;
chomp(@blogger);
$size = $#blogger;
for ( $count=0;$count<=$size;$count++ ) {
 $string = $wordpress[$count] . "|" . $blogger[$count];
 push(@combined, $string);
}
print "rewriteEngine on\n";
foreach ( @combined ) {
 s/\|/ /;
 s/^http:\/\/*.?\///;
 print "rewriteRule $_ [R=permanent,L]\n";
}
print "rewriteRule ^.*\$ http://food.paulcz.net [R=permanent,L]\n"; 
```

Run the script and output to a file.   The contents of this file can be added to a .htaccess file and will immediately start redirecting your traffic to the correct post on your new blog.   Any hits that don't match a mapped blog post will get redirected to the main page of your blog.

``` bash Terminal
perl rewrite.pl > rewrite.txt 
```

Inspect the file and if it looks correct you can append it to your `.htaccess` file.   If you have other redirect rules already in your `.htaccess` file you'll need to remove them or comment them out.

``` bash Terminal
cat rewrite.txt >> /var/www/html/.htaccess
```

There's no need to restart apache for a changed .htaccess file,  so you can immediately test that the redirects are working ...


1. [http://xesla.ro](http://xesla.ro) --> [http://food.paulcz.net/](http://food.paulcz.net/)
2. [http://xesla.ro/.../prickly-pear-syrup](http://xesla.ro/wordpress/cooking/prickly-pear-syrup) --> [http://food.paulcz.net/.../prickly-pear-syrup.html](http://food.paulcz.net/2012/08/prickly-pear-syrup.html)

Perfect!  we're working.


## Step 5.   More Things to do ... ?

This has taken care of almost everything I care about.  However I do want to be able to cancel my webhost subscription so I'll need to try and find a cloud type service to perform the redirects for me.    [Heroku](www.heroku.com) is probably the place I'll go for that.

