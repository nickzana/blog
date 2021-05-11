# Source code for Nick Zana's Blog
the `content` directory is in Markdown, so it can be browsed directly. Otherwise, a live version of the site can be found [here](https://blog.nickzana.dev/) and the Atom feed can be found [here](https://blog.nickzana.dev/atom.xml).

# Deployment
[zola](https://getzola.org) is the static site generator used. Simple install zola per their [instructions](https://www.getzola.org/documentation/getting-started/installation/) and run `zola serve` for a local version or `zola build` for the static site to be placed inside the `public` folder.

# Theme
I am using my own [fork](https://github.com/nickzana/zola-private-dev-blog) of the [simple-dev-blog](https://www.getzola.org/themes/simple-dev-blog/) theme. It intentionally strips out any external dependencies, JavaScript, or trackers.
