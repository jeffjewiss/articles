# Hosting Ghost Blogs with Dokku

I’m a big fan of the thinking behind [The Twelve Factor App](http://12factor.net) and deploying applications with tools and platforms like Heroku. I wanted to be able to leverage the power of a platform like Heroku for automating and managing deployments, and also take advantage of [Let’s Encrypt](https://letsencrypt.org/) for free, automated SSL certificates, and not have to pay per month for my experiments. Bonus points if I could setup a system that closely mirrors the Heroku setup, so if one of my projects does take off, there wouldn’t be many steps to switching it over to run on Heroku.

The next set of requirements for this deployment come from [Ghost](https://ghost.org), the blogging platform, which has a has a few quirks for a Heroku-style deploy that are worth knowing. So the goal is to have a server as a deployment target, which can be setup as a git remote, and when the Ghost install is sent to the server via `git push` the server builds the Node.js application and serves it using the production Ghost server.

## Enter Dokku

[Dokku]() is a docker-powered platform as a service that can run applications configured to run on Heroku. It’s a much more minimal platform, but for tiny projects or tiny teams it can suit your needs and is easy to install, maintain and upgrade.

