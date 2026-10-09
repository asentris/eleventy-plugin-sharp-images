# eleventy-plugin-sharp-images

An Eleventy plugin that brings the full capabilities of Sharp to your static sites, optimizing assets and improving website performance. Brought to you by [CodeStitch](https://codestitch.app)!

This plugin is a continuation of the now-abandoned [eleventy-plugin-sharp](https://github.com/luwes/eleventy-plugin-sharp) by [luwes](https://github.com/luwes/).


> [!TIP]
> A tutorial video covering all of the information in this README can be found [on YouTube](https://www.youtube.com/watch?v=scYFC1LRfPg)


## Table of Contents

-   [Features](#features)
-   [Installation](#installation)
-   [Configuration](#configuration)
-   [Usage](#usage)
    -   [Examples](#examples)
-   [VSCode Snippet](#vscode-snippet)
-   [How It Works](#how-it-works)
-   [Special Thanks](#special-thanks)

<a href="#features"></a>

## Features

1. Full Sharp integration, allowing for cropping, resizing, compressing, manipulating, and changing file types within your Eleventy project
2. Efficient caching mechanism to prevent regeneration of identical images within and between builds. Processed files and the cache manifest live on disk and can be persisted between deploys
3. Asynchronous processing - even when using non-asynchronous features (like Nunjucks Macros)

<a href="#installation"></a>

## Installation

1. Install the plugin:

    - Install a specific version directly from the public GitHub repository:
        - `npm install github:asentris/eleventy-plugin-sharp-images#v3.0.0`
    - Alternatively, install the latest commit from main:
        - `npm install github:asentris/eleventy-plugin-sharp-images#main`

2. Configure Eleventy:

```javascript
// eleventy.js

const eleventyPluginSharpImages = require("@asentris/eleventy-plugin-sharp-images");

module.exports = function (eleventyConfig) {

    // other plugins

    eleventyConfig.addPlugin(eleventyPluginSharpImages, {
        urlPath: "/assets/images",
        outputDir: "public/assets/images",
        autoRotate: true, // Automatically fix EXIF orientation issues
    });
};
```

> [!IMPORTANT]
> This plugin relies on specific HTML comments to process images. If these comments are removed or altered by minification before this plugin runs, it will cause errors. To prevent this, make sure to add any HTML minification plugins _after_ this plugin in your Eleventy configuration file. This ensures that image processing occurs before any minification takes place.


> [!CAUTION]
> `eleventy.js` only accepts one `module.exports`. Make sure you paste the plugin snippet above **inside** the current `module.exports`.

The plugin's only runtime dependency is `sharp`. Eleventy is provided by the site that installs this plugin.

<a href="#configuration"></a>

## Publishing

This package is distributed directly through GitHub using Git tags. No npm registry publication is required. Updating the version field in `package.json` is optional, but recommended for consistency.

- Commit and push your changes to main:

  ```
  git checkout main
  git pull origin main
  git merge dev main
  git push origin main
  ```

- Create and push a version tag:

  ```
  git tag --sort=-creatordate | Select-Object -First 2
  git tag -a v3.0.0 -m "Description."
  git push --tags
  ```

For each new release, create and push a new Git tag (e.g., v1.2.1). Existing tags should remain unchanged. Consuming projects can update by installing the new tag.

## Configuration

The plugin accepts the following configuration options:

-   `urlPath`: The prefix for generated image URLs in your built site
-   `outputDir`: The directory where processed images should be saved
-   `cacheStrategy`: Cache strategy to use ("auto", "content", or "stats"). Defaults to "auto"
-   `autoRotate`: Some images have a mis-match between their EXIF orientation and how the image is rendered. When this config option is enabled (Defaults to `true`), automatically apply EXIF orientation correction only when needed (i.e. when orientation ≠ 1)

<a href="#usage"></a>

## Usage

The plugin works by using the `{% getUrl %}` shortcode, supplying an image URL, and chaining filters that correspond to [Sharp's transformation options](https://sharp.pixelplumbing.com/api-output):

```html
<picture>
    <source srcset="{% getUrl "/assets/images/image.jpg" | resize({ height: 50, width: 50 }) | avif %}" media="(max-width: 600px)" type="image/avif"> <source srcset="{% getUrl
    "/assets/images/image.jpg" | resize({ height: 50, width: 50 }) | webp %}" media="(max-width: 600px)" type="image/webp"> <source srcset="{% getUrl "/assets/images/image.jpg" |
    resize({ height: 400, width: 400 }) | jpeg %}" media="(min-width: 601px)" type="image/jpeg"> <img src="{% getUrl "/assets/images/image.jpg" | resize({ height: 400, width: 400
    }) | jpeg %}" alt="Description of the image">
</picture>
```

In this example, we set up responsive image HTML with three `<source>` elements. Each source generates an image from `/assets/images/image.jpg`, using the `resize` filter to crop the image to specific dimensions, before outputting the image in AVIF, WebP, or JPEG format.

The processed image is then cached to prevent unnecessary regeneration.

Each Sharp transformation can be used as a filter, with options passed as an object. For example, to adjust the image position during resizing:

```nunjucks
{% getUrl "/assets/images/image.jpg" | resize({ height: 50, width: 50, position: "top" }) | avif %}
```

Please, make sure each transformation option has a value associated with its key. Failing to do so will result in errors being thrown when building the site:

**Incorrect**

```nunjucks
{% getUrl "/assets/images/image.jpg" | resize({ height: , width: 50 }) | avif %}
```

**Correct**

```nunjucks
{% getUrl "/assets/images/image.jpg" | resize({ height: 50, width: 50 }) | avif %}
```

<a href="#examples"></a>

### Examples

1. Resize image to set dimensions:

```nunjucks
{% getUrl "/assets/images/image.jpg" | resize({ height: 50, width: 50 }) %}
```

2. Resize image to set dimensions and convert to AVIF, with a quality value of 75:

```nunjucks
{% getUrl "/assets/images/image.jpg" | resize({ height: 50, width: 50 }) | avif({ quality: 75 }) %}
```

3. Resize image to set dimensions, cropped to the area of interest, and convert to AVIF:

```nunjucks
{% getUrl "/assets/images/image.jpg" | resize({ height: 50, width: 50, position: "attention" }) | avif %}
```

4. Rotate an image 90 degrees and convert it to grayscale:

```nunjucks
{% getUrl "/assets/images/image.jpg" | rotate(90) | grayscale %}
```

<a href="#vscode-snippet"></a>

## VSCode Snippet

For VSCode users, we've created a snippet that can help streamline your workflow when using the plugin:

```
    "eleventy-plugin-sharp-images Snippit": {
        "prefix": "respimg",
        "body": [
            "<picture class=\"${1}\">",
            "   <!--Mobile Image-->",
            "   <source media=\"(max-width: 600px)\" srcset=\"{% getUrl \"${2:/assets/images/placeholder.jpg}\" | resize({ width: ${3}, height: ${4} }) | avif %}\" type=\"image/avif\">",
            "   <source media=\"(max-width: 600px)\" srcset=\"{% getUrl \"${2:/assets/images/placeholder.jpg}\" | resize({ width: ${3}, height: ${4} }) | webp %}\" type=\"image/webp\">",
            "   <source media=\"(max-width: 600px)\" srcset=\"{% getUrl \"${2:/assets/images/placeholder.jpg}\" | resize({ width: ${3}, height: ${4} }) | jpeg %}\" type=\"image/jpeg\">",
            "   <!--Tablet Image-->",
            "   <source media=\"(max-width: 1024px)\" srcset=\"{% getUrl \"${2:/assets/images/placeholder.jpg}\" | resize({ width: ${5}, height: ${6} }) | avif %}\" type=\"image/avif\">",
            "   <source media=\"(max-width: 1024px)\" srcset=\"{% getUrl \"${2:/assets/images/placeholder.jpg}\" | resize({ width: ${5}, height: ${6} }) | webp %}\" type=\"image/webp\">",
            "   <source media=\"(max-width: 1024px)\" srcset=\"{% getUrl \"${2:/assets/images/placeholder.jpg}\" | resize({ width: ${5}, height: ${6} }) | jpeg %}\" type=\"image/jpeg\">",
            "   <!--Desktop Image-->",
            "   <source media=\"(min-width: 1024px)\" srcset=\"{% getUrl \"${2:/assets/images/placeholder.jpg}\" | resize({ width: ${7}, height: ${8} }) | avif %}\" type=\"image/avif\">",
            "   <source media=\"(min-width: 1024px)\" srcset=\"{% getUrl \"${2:/assets/images/placeholder.jpg}\" | resize({ width: ${7}, height: ${8} }) | webp %}\" type=\"image/webp\">",
            "   <source media=\"(min-width: 1024px)\" srcset=\"{% getUrl \"${2:/assets/images/placeholder.jpg}\" | resize({ width: ${7}, height: ${8} }) | jpeg %}\" type=\"image/jpeg\">",
            "   <img src=\"{% getUrl \"${2:/assets/images/placeholder.jpg}\" | resize({ width: ${7}, height: ${8} }) | jpeg %}\" alt=\"${9}\" width=\"${10}\" height=\"${11}\" loading=\"${12:lazy}\" decoding=\"async\" ${13:aria-hidden=\"true\"}>",
            "</picture>"
        ],
        "description": "eleventy-plugin-sharp-images Snippit"
    }
```

Simply, place this into your snippets file for the HTML language in VSCode. More information on installing and using the snippet can be found within the appropriate section on the [YouTube tutorial video](https://youtu.be/scYFC1LRfPg?si=pMuDRu3FbiCNq8Ac&t=1532).

<a href="#how-it-works"></a>

## How It Works

To support environments where async features aren't allowed (like processing an image in a Nunjucks Macro), the shortcode doesn't directly generate the image. Instead, it creates a comment with a JSON configuration object. At the end of an Eleventy build, a Transform uses a regex to find all instances of these comments and process the images.

The configuration is hashed, and the file is renamed to include this hash. If an image path or its transformations change, the hash/filename will change, invalidating the cache. The manifest is written to `.cache/eleventy-plugin-sharp-images.json`. Persisting that directory and `outputDir` between deploys skips reprocessing.

<a href="#special-thanks"></a>

## Special Thanks

-   [luwes](https://github.com/luwes/) for building the original [eleventy-plugin-sharp](https://github.com/luwes/eleventy-plugin-sharp)
-   [Multiline Comment](https://multiline.co/) for their article on [using Eleventy transforms to render asynchronous content inside Nunjucks macros](https://multiline.co/mment/2022/08/eleventy-transforms-nunjucks-macros/)
