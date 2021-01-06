---
title: Getting Started
weight: 1
---

This guide will show you how to setup a local environment for you to edit, create, or update content!

Please note, if you're wanting to make a simple edit, you can always submit a quick pull request by utilizing the edit button on the file in question directly on the github repo online. 

## Installing Hugo

Since we are using Hugo, getting your local site up and running is fairly simple. 

### OS Independent

Since Hugo is cross-platform, and OS choice is far from uniform in this community, I won't go into how to get Hugo functioning on your OS of choice. Follow the [Getting Started](https://gohugo.io/getting-started/quick-start/). It's not difficult. 

Once you can successfully get a version number from the command `hugo version`, you're ready to continue here. 

### Fork and Clone Operations

First think you'll want to do is Fork the [Github Repository](https://github.com/r-selfhosted-wiki/wiki) for the /r/selfhosted wiki. You'll be working within a repo that syncs with your own accounts' fork of the repository. 

Since we use a theme that has its own [Github Repository](https://github.com/matcornic/hugo-theme-learn), there is an extra flag we must add to our `git clone` command. 

1. First, clone the forked repo into your local machine. the "recurse-submodules" flag should allow you to automatically pull in the git repo for the theme, as well.

    `git clone --recurse-submodules https://github.com/{YOUR_USER_NAME}/wiki.git`  -- Be sure to modify this url to match your actual username and git repository name. 

2. Move to the directory that was just cloned and make sure the `themes/hugo-theme-learn/` folder has content. 

    `cd wiki && ls themes/hugo-theme-learn`

3. Once confirmed, copy the example `config.toml` file. Unless you're doing some abstract hosting environment for your local developement machine, this should work as-is. 

    `cp config.toml.example config.toml`

4. Run the server locally with the Hugo's server command

    `hugo serve`

You should see some output about the success/launch of the local server, similar to below: 

```
$ hugo serve
  Start building sites … 
  
                     | EN  
  -------------------+-----
    Pages            | 22  
    Paginator pages  |  0  
    Non-page files   |  0  
    Static files     | 75  
    Processed images |  0  
    Aliases          |  0  
    Sitemaps         |  1  
    Cleaned          |  0  

  Built in 84 ms
  Watching for changes in /home/kmisterk/wiki/{archetypes,content,data,layouts,static,themes}
  Watching for config changes in /home/kmisterk/wiki/config.toml
  Environment: "development"
  Serving pages from memory
  Running in Fast Render Mode. For full rebuilds on change: hugo server --disableFastRender
  Web Server is available at http://localhost:1313/ (bind address 127.0.0.1)
  Press Ctrl+C to stop
```


Navigate your browser to `http://localhost:1313` and you should see the site live on your local machine. 

You're now ready to start adding and/or editing content. 