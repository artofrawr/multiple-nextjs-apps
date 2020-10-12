## !!! Please note !!!
(10/2020) This proof of concept is outdated. I'll try to update its dependencies if and when I find a chance. In the meantime please treat it as a theoretical illustration of the concept.

# multiple-nextjs-apps
An example for running multiple Next.js apps on the same domain.


## 1. Setting asset prefix
[Setting assetPrefix](https://nextjs.org/docs#cdn-support-with-asset-prefix) will prefix all scripts that Next.js loads. E.g. `/app1/_next/...`.

This can also be [set dynamically](https://nextjs.org/docs#dynamic-assetprefix) in the server.js, which is useful when changing the assetPrefix based on incoming requests for example.

```js
// in the examples apps the assetPrefix is set in server.js
app.setAssetPrefix('/app1');
```

Only setting the asset prefix dynamically or based on environment variables allows for a local development workflow that's mostly unchanged.

## 2. Nginx config
In the nginx config two things are handled: page requests and asset prefixed requests are being sent to the correct app.

For example:
- `/` -> app1
- `/other-page` -> app2
- `/app1` -> app1
- `/app2` -> app2

## 3. Rewriting the request URL for Next.js
Once the request arrives at the Next.js server, it's being rewritten so for Next.js it looks as if it's a completely normal request.

```js
// in server.js
server.use(function(req, res, next) {
  req.url = req.originalUrl.replace('app1/_next', '_next');
  next();
});
```

## 4. Static URLs
The only thing affecting the development workflow once this setup is completed, is how static assets are handled. The `assetPrefix` setting has no effect whatsoever on `/static`. So any references in the code to static urls will have to be manually prefixed, for example by the use of a helper function (e.g. `staticPath('images/image.png')`), which will turn the url into the prefixed path when necessary (e.g. `/app1/static/images/image.png`).
