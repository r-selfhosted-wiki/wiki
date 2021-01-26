---
Title: What is Self Hosting?
---

**The act of providing or serving digital content or an online service typically delivered by a business.**

It is generally served locally, as hosting it on a VPS or other internet residing machine is not "hosting" it yourself. There is still a middleman, and that is the owner/operator of that internet residing machine.

---

One of the easiest things to self host with the lowest barrier to entry is a website. For the most basic website of your own, all you need is a domain name and a webserver. Then you throw a few lines of HTML in a file and you have yourself a "website". With a service like LetsEncrypt, securing the site with an SSL certificate is easy too.

A lot of different services that you can self-host are "websites". There are dynamic sites with robust content management systems like joomla, drupal, wordpress, or b2evolution. There are forums like phpBB, MyBB, vBulletin, discourse, etc. Knowledge bases like DokuWiki, MediaWiki, BookStack, or Gollum are also websites. These websites only require a webserver, an interpreter ( php ), and a database ( sqlite, postgresql, mysql ).

Just about everything these days has a web UI or frontend to make things easier. HTTP/HTML/JS are well understood standards that are ubiquitous. There are many libraries for converting or presenting your content in a web friendly way for almost all programming languages you can learn.

It can be hard for someone unfamiliar to find a difference in the "website" frontend and the content backend. Sometimes the difference is almost non-existent. Sometimes there are many layers and systems working behind the scenes to make it happen.

It may be better to say that everything can be "accessed" through a website, even if it isn't one *per-se*. And if it can't, there's probably a separate piece of software that makes a web-ui for it.

Examples of services with a web-ui or separate web-based front-ends are: bit-torrent clients like qbittorrent/transmission, media streaming servers like jellyfin/navidrome, file synchronization services like nextcloud/owncloud/seafile, communication services like synapse/inspircd/jabberd/mumble, and many more.

Other services use the server-client model where the entire package is in two parts. The server part that runs at all time to serve content and the client part that connects to have content served to it. Examples are: game servers like rust/minecraft/factorio, ftp servers, email servers, and more.
