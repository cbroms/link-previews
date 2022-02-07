# Link Previews

Link previews everywhere.

When added to a webpage, hovering links displays a wikipedia-style dynamic page preview.

**This project is currently being rewritten, find [the previous version here](https://github.com/cbroms/link-previews/tree/v0.1.0)**.

## Usage

There are two parts to this project:

- A script that inserts page previews on your webpage, written as a web component.
- A serverless worker function that fetches and serves the links' meta information.

The worker is required because CORs prevents cross-origin client requests. For example, trying to directly `fetch` a url like `https://matrix.org` from your webpage will fail since most servers are configured to reject cross-origin requests. As a result, it's necessary to make the request outside the browser.

Since the task of fetching a page and parsing out the meta information is so simple, it can be accomplished with a single serverless function. I'm using [Cloudflare Workers](https://workers.cloudflare.com/) but you could probably rewrite it for some other service like AWS Lambda.

### Deploy the worker

Follow the instructions in [the worker's readme](worker/README.md) to deploy the worker. Once complete, you should have a url like `https://worker.[something].workers.dev`, or a custom domain if you've opted to set one up.

### Add the script and instantiate

The latest build can be found in [`client/dist`](/client/dist/).

Add the script to the end of your site's `body`:

```html
<script src="hyperfov-link-previews.js"></script>
```

Then, call `linkPreview` for each link you'd like to add a preview to:

```html
<html>
  <head>
    <title>Link Previews</title>
  </head>
  <body>
    <a id="myLink" href="https://hyperfov.com">A cool link</a>

    <script src="hyperfov-link-previews.js"></script>
    <script>
      // having loaded the script, add a link preview to the <a> tag of interest
      linkPreview("#myLink", {
        backend: "https://link-to-worker.workers.dev",
      });
    </script>
  </body>
</html>
```

### Options

You can customize the preview element through the `options` in the constructor:

```js
linkPreview("#myLink", {
  backend: "https://link-to-worker.workers.dev",
  template: "#my-cool-template", // a custom template for rendering the preview
  position: "below", // "below" or "above" the link, or "follow" the cursor
  title: "An interesting link", // setting the title or desc overrides the link's data
  description: "I think is worth clicking",
});
```

Here's the full list of options:

| Option        | Value                                                                                                                                                                          | Default   | Required? |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | --------- | --------- |
| `backend`     | A string with the URL of the deployed worker (see [Deploy the worker](#deploy-the-worker).)                                                                                    | `null`    | `false`   |
| `template`    | The selector of the template element to use to render the preview (see [Custom styles and markup](#custom-styles-and-markup).) The default `"basic"` uses a provided template. | `"basic"` | `false`   |
| `fetchUrl`    | Fetch the url's content from the worker?                                                                                                                                       | `true`    | `false`   |
| `position`    | Where the preview will be placed relative to the link. `"below"`, `"above"` or `"follow"` to follow the cursor                                                                 | `"below"` | `false`   |
| `title`       | The preview title. Overrides the title produced by the worker if it finds one.                                                                                                 | `null`    | `false`   |
| `description` | The preview description. Overrides the description produced by the worker if it finds one.                                                                                     | `null`    | `false`   |
| `url`         | The url of the link. Overrides the provided element's `href`.                                                                                                                  | `null`    | `false`   |
| `img`         | The preview image `src`. Overrides the image `src` produced by the worker if it finds one.                                                                                     | `null`    | `false`   |

### Custom previews

The link preview is totally customizable through an [html template](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/template) with [slots](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/slot). The template that's used by default can be found at [`client/templates/basic.js`](/client/templates/basic.js).

Add a custom template to your page (the `template` element is invisible, so this can be anywhere in the document):

```html
<template id="link-preview-template">
  <style>
    .wrapper {
      padding: 20px;
      background-color: lightsalmon;
    }
  </style>
  <div class="wrapper">
    <slot name="title">my title</slot>
    <div><slot name="description">my description</slot></div>
    <div><slot name="url">my url</slot></div>
  </div>
</template>
```

The styling and markup is totally up to you, though you must ensure styles are either inline or included in a `style` tag in the template. The link preview component will look for a slots `title`, `description`, `url`, and `img` to insert content. Data that doesn't have corresponding slot in the template won't be displayed.

Once you've created a template, pass it in as an option when instantiating your link previews:

```js
linkPreview("#myLink", {
  backend: "https://link-to-worker.workers.dev",
  template: "#link-previewtemplate",
});
```

## Development

### Worker

Follow the directions to start the worker in [the worker's readme](worker/README.md).

### Client

To run the client, enter the `client` directory and install the dependencies:

```
npm install
```

Then run the development server:

```
npm run dev
```

You should now be able to navigate to `localhost:9000` in the browser to try out the examples.

To build the client:

```
npm run build
```
