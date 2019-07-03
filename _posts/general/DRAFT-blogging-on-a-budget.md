About a year ago, I set about to redesign my blog. I had used [ghost](https://blog.ghost.org/) for the first implementation of my blog. It was a pretty nice platform, but running it on the cheap was always an issue(their hosted platform isn't cheap). Obviously, paying for hosting on a variety of platforms is an option, but none that I found were inexpensive and given that I'm not making a penny on the site currently...

Upon signing up for Azure, one is given a number of free items. One of those is 10 free webapps. I set it up on an Azure website utilizing the [Ghost-Azure](https://github.com/felixrieseberg/Ghost-Azure) tool. This is a slick tool that sets everything up nicely for you, but I found that I didn't understand it very well and it became clear pretty quickly that upgrading with it wasn't going to be easy. The version of ghost I was running, quickly became out-dated and I knew eventually I would want to find something else.

In addition, while Azure provides some free websites when you sign up, the sites are limited. For instance, it is only free if you don't use a custom domain. There were a number of different options I explored to get around this limitation, like using Cloudflare in front of the webapp, but either a) I never properly figured out how to set them up or b) the setup that I was going for wasn't possible with their free offerings. Long term, that ruled it out for me. There were a few other annoyances such as setting up SSL with a custom domain and getting posts routed to properly while trying to setup a custom domain through Google Domains and having it route properly to the Azure app.

All of these factors, made it clear to me that I would have to look down a different path for a better free long term solution.

Initially I created an [ASP.NET Core solution](https://github.com/peinearydevelopment/peinearydevelopment) for a blog. This gave me the ability to start kicking around some design ideas as well as to play around with a few proof-of-concept ideas. The thought being that if I would need to run the blog on a managed platform, the cheapest solution would be to run it on a Linux box. After completing this and looking around for a way of hosting the blog, I quickly grew tired of the exercise. There are so many solutions out there, none of which are super cheap and I found it very hard to evaluate the plusses and minuses of each. I had heard of [GitHub Pages](https://pages.github.com/) before, but never really had the time to investigate them. I figured now would be an opportune time. What I found was quite eye-opening, enjoyable and affordable.

GitHub Pages serves static files. As stated on their homepage, that can be as easy as creating an `index.html` file in your repository(named with a certain convention, also detailed very nicely in their documentation). I was looking to do a bit more and their most frictionless way of doing so is through [Jekyll](https://jekyllrb.com/). Jekyll is a static site generator built with ruby. All of that mumbo-jumbo means that I could create all of my blog posts in markdown and style the whole thing with scss. The jekyll 'engine' will take those files and create static html files and the routing in between the files. At this point, I had one main issue. I don't have ruby on my computer and didn't want to go through all the setup steps to get it working properly. I'm sure if I was more familiar with this platform, I could just create the files, push them to the GitHub repo and have things work, but I wanted to iterate on a number of different areas of the site development and wanted to be able to do so locally. This is where Docker really came in handy.

<aside>This post isn't about Docker which is good because I don't know too much about it and am exploring it more at the moment. I would just like to take a moment to detail what I did with Docker for this blog to point the curious reader in the right direction if they wanted to try this out as well. I have [Docker Desktop](https://www.docker.com/products/docker-desktop) installed on my Windows 10 machine. I then created a [`docker-compose.yml`](https://github.com/PinaryDevelopment/blog/blob/master/docker-compose.yml) file at the root of my GitHub repository. From that directory, with Docker Desktop installed, I'm now able to run the command `docker-compose up` from a powershell window. That will cause the jekyll engine to create all the static files in a `_site` folder, launch a server and put the website on `http://localhost:4000`.</aside>

At this point the site is completely empty, but I utilized the Jekyll docs to conform to a recommended project structure. Along the way, I also referenced a few big name projects' repos to try to understand other possibilities and approaches to common problems.

Let's walk through a few of those.

# Secure Custom Domain

As detailed in the GitHub pages documents, this was relatively easy to setup.
- Buy a domain. I bought my through `domains.google.com` and I believe it costs me around $12/yr.
- Configure a DNS entry with your domain provider.
- Add a file called `CNAME` at the root of the repository whose only contents consist of the site domain. For me this was `blog.pinarydevelopment.com`
- Then in the `Settings` tab in GitHub, `Options` subtab, you need to add the same domain to the `Custom domain` section and click the checkbox for `Enforce HTTPS`
- The `Enforce HTTPS` isn't available immediately. I had to check back a few times before I was actually able to check it off.

And thats it. I know its a few steps, but the whole time, GitHub pages manages getting and renewing the certificates for the site to ensure it is served over HTTPS and it doesn't cost a thing.

# Sass-y Styling

Through a number of projects that I've worked on in the past, I've become comfortable working with scss files for my site's styling needs. I like the ability to utilize a number of its functions to compose different shared stylings together. This wasn't a big deal to do at the end of the day, but it took me a little stumbling to get there. As Jekyll is a static files generator, the actual reference in the html file to the stylesheet needs to point to a css file(this gets generated by Jekyll, so you won't see it the checked-in source files). In my default layout, I have the following reference `<link rel="stylesheet" href="/main.css" />`. The `_config.yml` file specifies the sass directory, but the main entry point for the sass styles isn't there. In order for this to work, I had to put a `main.scss` file in the root of project. As can be [seen also](https://github.com/PinaryDevelopment/blog/blob/master/main.scss), it is a little strange. It references the `site.scss` file that's in the sass directory as if they are adjacent to each other. In addition it needs to have the special `---` enclosed ['front matter'](https://jekyllrb.com/docs/front-matter/) header.

# Limitations

Just to be clear though, not all is as rosy as presented above. The above was relatively easy to accomplish, smooth to setup and affordable to maintain. The fact that it is relying on a static site generator inherently introduces some limitations. Two big areas that come to mind are giving the ability for people who visit your site to leave comments on posts and the ability to search across the site for finding posts of interest. There are a few different potential approaches to enable these pieces of functionality in the site though.

### Comments

There are a few plugins/extensions that have been written to accomplish the above. [Disqus](https://disqus.com/) is a service that can be signed up for and then with the addition of a script tag on each page, your site magically has comments. There are many pluses to this approach. It is easy to setup, free and handles all the storage and logic around infinitely nested comments/replies, etc. Once a user signs up with it as well, they are able to track, manage all of the activity related to their comments across the web in one place. The downsides are that it is a bit clunky/slow to load and not straightforward to skin inline with your brand.

Some others that I found were: [GitHub Comments](https://github.com/wireddown/ghpages-ghcomments) and [StaticMan](https://staticman.net/). I tried to setup StaticMan, but ran into many issues to the point that I gave up and moved on. Another option that I explored is to create my own [GitHub App](https://github.com/PinaryDevelopment/GitHubApp.Commentary) that would, through the help of an Azure Function, create a PR to my repository anytime someone wanted to comment on a post. This would provide me the ability to screen the comments before they got added to any post and would provide me a bit more control over the comments section itself. The downside here is that it is a dev effort that I haven't had the ability to complete to date :(.

### Search

http://www.jekyll-plugins.com/plugins

# Additional Resources

There were a number of other things that I tried to do with the site that took some serious fiddling. I found a few resources along the way that were helpful though. Jekyll is a ruby program, so there were occasions that I found myself needing to do something in Ruby(that I have no experience with). For instance, I needed to utilize Ruby to format some dates and found [this very useful guide](https://hackhands.com/format-datetime-ruby/). The markup syntax that Jekyll utilizes is called liquid. The most useful reference I could find on this came from [Shopify](https://shopify.github.io/liquid/).
