---
title: Build Drag&Drop upload components in no time using Dropzone.js
date: 2021-03-03T23:00:00+00:00
hero: "/images/hero-6.jpg"
excerpt: 'A simple guide on how to build feature rich upload components inside vue.js.. '
timeToRead: 5
authors:
- alexander-grossmann

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

I will use the vue wrapper of dropzone, called vue-Dropzone [vue-Dropzone](https://rowanwins.github.io/vue-dropzone/docs/dist/#/installation "vue-dropzone").

First of all we are adding dropzone to our project.

``` js
    npm install vue2-dropzone
```

After the installation, you have full access to the dropzone.js functionality documented under: , and it could be imported like any other vue-component.

Next, we will set up the dropzone component, this should look something like this:

![](/images/d-d-component.png)

If The Image was uploaded successfully, it will be displayed like on the left, with the image name on Huver.

Since we do not allow duplicated images on our S3 Bucket, adding the same Image twice will lead to an error (right), the image will be marked and on hover, the error message will be displayed.

```js
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
          files: new FormData(),
          baseURL: "youApiUrl",
          dropOptions: {
            url: baseURL + "yourEndpoint",
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
      methods: {
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
        },
      }
    };
    </script>
```

In the above code, we first import the dropzone component and display a label given as a prop above it.

The options passed to the component are defined in the dropOptions object.

With this options set, our component will send a formData Object to the given URL, got remove links to delete uploaded images again. The accept property could be used to define the accepted file types via this filter function, with maxFileSize we are not accepting files bigger than 3MB. The headers are set according to the needs of the API.

You can also intercept Dropzone at different stages and execute additional code. For example, I'm appending the location where the image should be saved to the request, by simply listening to the _@vdropzone-sending_ event. Again you can find the whole list of supported events in the  [dropzone.js documentation](https://www.dropzonejs.com/ "dropzone.js docs").

This was the first part of my series on how to securely upload anything to Amazon S3. In the next part, I'm going to explain how to stream the image through a .net Core 3.1 Api and store a reference of the Image inside it.