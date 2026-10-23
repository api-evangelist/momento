---
title: "Stop counting indexes"
url: "https://www.gomomento.com/blog/stop-counting-indexes/"
date: "2026-08-27"
author: "Allen Helton"
feed_url: "https://www.gomomento.com/feed"
---
Fifty extra indexes don't slow down your writes, but one vector field slows them down 11x. I wanted to figure out why valkey-search behaves this way, and it wasn't what I expected.
