---
title: Uploading images to Amazon S3
date: 2020-03-01T23:00:00+00:00
hero: "/images/hero-6.jpg"
excerpt: A simple guide on how to securly upload images to your amazon S3 Bucket from
  an vue.js frontend and an .net Core 3.1 Backend.
timeToRead: 10
authors:
- Alexander Grossmann

---
In this guide, i want to show you how to build a Drag&Drop inside your vue application and upload the dragged Images to Amazon S3 in a secure way.

For Uploading the images we are going to use vue dropzone, and to make the upload process more secure the Images will at first be uploaded to a .net server and from there it will be streamed to your S3 Bucket.

This will improve the overall security of your application and your AWS account since your sensitive login info is not saved inside your frontend where it could be easily exploited. Instead, it is saved in your backend application where an attacker could not access it.

This article will be split into three sections.

In Section One, I will describe how to set up the UI.

Section two will all about handling the images inside that backend and forwarding them to your S3 Bucket.  You can skip this if you want to use another server technology.

In the last section, you will learn everything you need to know about creating your S3 Bucket and saving the images to it.

# Building the UI

The Images are going to be uploaded using dropzone.js, this is an amazing js library providing feature-rich drag&drop functionality with upload status etc. in no time.

### Install and setup Dropzone.js

I will use the vue wrapper of dropzone, called vue-Dropzone [https://rowanwins.github.io/vue-dropzone/docs/dist/#/installation](https://rowanwins.github.io/vue-dropzone/docs/dist/#/installation "vue-dropzone").

First of all we are adding dropzone to our project.

    npm install vue2-dropzone

After the installation, you have full access to the dropzone.js functionality documented under: , and it could be imported like any other vue-component.

Next, we will set up the dropzone component, this should look something like this: 

![](/images/d-d-component.png)

If The Image was uploaded successfully, it will be displayed like on the left, with the image name on Huver. 

Since we do not allow duplicated images on our S3 Bucket, adding the same Image twice will lead to an error (right), the image will be marked and on hover, the error message will be displayed.

    <template>
      <div>
        {{ label }}
        <vue-dropzone
          id="drop1"
          ref="myVueDropzone"
          :options="dropOptions"
          @vdropzone-sending="appendLocation"
          @vdropzone-success="handleResponse"
        ></vue-dropzone>
      </div>
    </template>
    
    <script>
    import ImageRepository from "../../Repository/ImageRepository";
    import "vue2-dropzone/dist/vue2Dropzone.min.css";
    import vueDropzone from "vue2-dropzone";
    
    export default {
      name: "dropZone",
      components: { vueDropzone },
      props: {
        label: {
          type: String
        },
        location: { type: String, required: true }
      },
      data() {
        return {
          selectedImages: [],
          images: [],
          files: new FormData(),
          file: "",
          baseURL: Repository.defaults.baseURL,
          dropOptions: {
            url: Repository.defaults.baseURL + "images",
            thumbnailWidth: 150,
            addRemoveLinks: true,
            maxFilesize: 3,
            accept: function(file, done) {
              console.log(file);
              if (
                file.type.toLowerCase() != "image/jpg" &&
                file.type.toLowerCase() != "image/gif" &&
                file.type.toLowerCase() != "image/jpeg" &&
                file.type.toLowerCase() != "image/png"
              ) {
                done("Invalid file");
              } else {
                done();
              }
            },
            headers: {
              "Cache-Control": null,
              "X-Requested-With": null,
              withCredentials: true
            }
          }
        };
      },
      created() {
        crudEventBus.$on("clearImages", info => {
          this.removeAllFiles();
        });
      },
      methods: {
        removeAllFiles() {
          this.$refs.myVueDropzone.removeAllFiles();
        },
        appendLocation(file, xhr, formData) {
          formData.append("path", this.location);
        },
        handleResponse(file, response) {
          console.log(response);
          var Image = {
            key: response.key,
            imageId: parseInt(response.id),
            bucket: this.location
          };
          this.selectedImages.push(Image);
          this.$emit("input", this.selectedImages);
          this.$emit("uploaded", this.selectedImages);
          this.selectedImages.length = 0;
        },
      }
    };
    </script>

In the above code, we first import the dropzone component and display a label given as a prop above it. 

# This is a primary heading

Do they have the resources necessary to execute on their ideas? Or are they constantly under pressure to pluck only the lowest-hanging fruit through bare minimum means, while putting their greatest ambitions on the back-burner?

> Blockquotes are very handy in email to emulate reply text.
> This line is part of the same quote.

But it takes more than good ideas to build and grow a business. It takes people to bring them into reality. Are those people collaborating and sharing their expertise, or are they in conflict and keeping it to themselves?

> This is a very long line that will still be quoted properly when it wraps. Oh boy let's keep writing to make sure this is long enough to actually wrap for everyone. Oh, you can _put_ **Markdown** into a blockquote.

These are the circumstances that suffocate creativity and destroy value in an organization. That’s why I knew that if I was going to start a company, our first product would have to be the company itself. These are the circumstances that suffocate creativity and destroy value in an organization. That’s why I knew that if I was going to start a company, our first product would have to be the company itself.

## This is a secondary heading

```jsx
import React from "react";
import { ThemeProvider } from "theme-ui";
import theme from "./theme";

export default props => (
  <ThemeProvider theme={theme}>{props.children}</ThemeProvider>
);
```

These are the circumstances that suffocate creativity and destroy value in an organization. That’s why I knew that if I was going to start a company, our first product would have to be the company itself. These are the circumstances that suffocate creativity and destroy value in an organization. That’s why I knew that if I was going to start a company, our first product would have to be the company itself. These are the circumstances that suffocate creativity and destroy value in an organization. That’s why I knew that if I was going to start a company, our first product would have to be the company itself. These are the circumstances that suffocate creativity and destroy value in an organization. That’s why I knew that if I was going to start a company, our first product would have to be the company itself.

***

Hyphens

***

Asterisks

***

Underscores

These are the circumstances that suffocate creativity and destroy value in an organization. That’s why I knew that if I was going to start a company, our first product would have to be the company itself. These are the circumstances that suffocate creativity and destroy value in an organization. That’s why I knew that if I was going to start a company, our first product would have to be the company itself.

Do they have the resources necessary to execute on their ideas? Or are they constantly under pressure to pluck only the lowest-hanging fruit through bare minimum means, while putting their greatest ambitions on the back-burner?

Emphasis, aka italics, with _asterisks_ or _underscores_.

Strong emphasis, aka bold, with **asterisks** or **underscores**.

Combined emphasis with **asterisks and _underscores_**.

Strikethrough uses two tildes. ~~Scratch this.~~

1. First ordered list item
2. Another item
   ⋅⋅* Unordered sub-list.
3. Actual numbers don't matter, just that it's a number
   ⋅⋅1. Ordered sub-list
4. And another item.

⋅⋅⋅You can have properly indented paragraphs within list items. Notice the blank line above, and the leading spaces (at least one, but we'll use three here to also align the raw Markdown).

⋅⋅⋅To have a line break without a paragraph, you will need to use two trailing spaces.⋅⋅
⋅⋅⋅Note that this line is separate, but within the same paragraph.⋅⋅
⋅⋅⋅(This is contrary to the typical GFM line break behaviour, where trailing spaces are not required.)

* Unordered list can use asterisks
* Or minuses
* Or pluses