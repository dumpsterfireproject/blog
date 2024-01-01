---
title: "Gatsby Path Prefix and Docker Build"
date: 2022-01-05
slug: "/gatsy-path-prefix"
description: Quick note on path prefix
tags:
  - Gatsby
---
Recently I needed to containerize a Gatsby site and deploy it to a kubernetes cluster.  The cluster had an
ingress controller that mapped the /docs path to the Gatsby service. I just wanted to take a quick minute and
document the process here.  I utilized the [pathPrefix](https://www.gatsbyjs.com/docs/how-to/previews-deploys-hosting/path-prefix/)
in Gatsby so that the service would host the website at the /docs path.

```javascript
const config = {
  gatsby: {
    pathPrefix: '/docs',
    ...
```
I then created the following Dockerfile to build the image that would be deployed to kubernetes. By running
the gatsby build within the container, I was able to run this in our CI/CD pipeline without having to worry
about any conflicts with any other jobs in regards to node versions or other dependencies.
```dockerfile
FROM node:16.13.0-apline3.14

WORKDIR /app

RUN apk add --no-cache python3 make gcc g++ util-linux
RUN npm -g install gatsby-cli@4.0.0
COPY . .
RUN yarn
RUN gatsby build --prefix-paths

CMD ["gatsby", "serve", "--verbose", "--prefix-paths", "-p", "80", "--host", "0.0.0.0"]
```
The one mistake that I made along the way was that I added the `--prefix-paths` argument to the gatsby serve command,
but forgot to add it to the gatsby build command.  When I deployed the container, the links in the menu options
and the pervious and next links at the bottom of the page did not include /docs in the path (I was using the
[gatsby-gitbook-starter](https://www.gatsbyjs.com/starters/hasura/gatsby-gitbook-starter) theme). Once I added the
`--prefix-paths` argument to the gatsby build command and redeployed, everything worked as expected.
