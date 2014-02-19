Using pelican for serving this blog
###################################
:date: 2014-02-19 15:40
:author: mrunge
:category: fedora
:slug: deprecating-wordpress

For my own blog, I recently deprecated wordpress, in favor of using pelican. 
The package was recently added to Fedora. It's quite simple and does not
require php, mysql, and a memcached running on a webserver. 

In the past, the number of attempts to get into this host increased 
significantly. During Linux-Tag-Oldenburg last year, we had a wordpress
installation to serve the schedule. It just failed when more persons tried
to edit the same document. 

Pelican uses structured text files, which are being processed once to html 
files. This means, one can use awesome tools like vim and git.
