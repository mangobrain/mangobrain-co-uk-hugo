---
title: "It's alive - again!"
date: 2021-12-17
comments: true
categories: update
---
My on-again, off-again relationship with having my own website is, at least for
the time being, officially on again. I've moved my domain name to a cheaper
name service, migrated the site itself from [Jekyll](https://jekyllrb.com/) to
[Hugo](https://gohugo.io/), and am hosting the website itself to
[Vercel](https://vercel.com/) on their free "Hobby" tier.

Why have I done these things?

Well, there's nothing wrong with Jekyll _per se_, but being implemented in Ruby
with plugins as Gems means it doesn't always play nicely with Linux package
managers: installing everything you need and keeping it up-to-date can be a
pain; I don't really want to have to deal with things like Gemfiles in the root
of the repository; and good luck if you want continuous deployment but the
build environment on your host of choice differs from what you have locally.
It's not _quite_ as bad as [Python](https://xkcd.com/1987/), but it's not far
off. Hugo is implemented in Go, and has built-in support for all the
functionality I need: installing it locally was as simple as
`dnf install hugo`, everything builds into a `public/` subfolder for static
deployment, or you can use continuous deployment with any provider that has the
same version of Hugo available (or newer).

Vercel allows me to set up a "project" linked directly to a
[GitHub repository](https://github.com/mangobrain/mangobrain-co-uk-hugo),
supports custom domains & SSL (with non-broken Let's Encrypt support, which is
more than I can say for my previous hosting provider), and has a free hosting
tier which I'm unlikely to exceed the usage limits on any time soon, if ever.
So I can just make changes, push them, then wait a few minutes whilst they run
Hugo on their end and deploy the changes automatically.

Magic!
