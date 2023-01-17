- Start Date: 2023-01-17
- RFC PR: (leave this empty)

# Summary

Parse and store more important meta information from an uploaded media file.

# Motivation

It can be very useful to store more meta informations for a specific file at the cms-side. Some content-types attributes for uploaded files are already present, but empty (e.g. width and height of a video file). Other information (like audio duration, number of channels, bitrates, ...) were already requested:
- [https://feedback.strapi.io/feature-requests/p/add-duration-to-audio-media](https://feedback.strapi.io/feature-requests/p/add-duration-to-audio-media)
- [https://forum.strapi.io/t/get-media-mp3-duration/1224](https://forum.strapi.io/t/get-media-mp3-duration/1224)

These information can be useful not only to query with the REST API and display on a UI, but also cms internally. Furthermor this could help improving the performance of the media library, which can get really slow, because currently the media assets get loaded and displayed entirly (instead of a thumbnail of it, also address in this [feature request](https://feedback.strapi.io/customization/p/create-a-thumbnail-of-an-uploaded-video), but that's another topic).

# Detailed design

I already tried to implement it very quickly as prove of concept. Therefor I used the node packages [ffprobe](https://www.npmjs.com/package/ffprobe) and [ffprobe-static](https://www.npmjs.com/package/ffprobe-static), which can read out many informations of a media file. There are probably other solutions which works as well or even better.

# Example

For the prove of concept I added some attributes to the content-type schema of _file_ in the [strapi-plugin-upload](https://github.com/strapi/strapi/blob/main/packages/core/upload/server/content-types/file/schema.js):

``` javascript
'use strict';

const { FOLDER_MODEL_UID } = require('../../constants');

module.exports = {
	...,
  attributes: {
	...,
    mime: {
      type: 'string',
      configurable: false,
      required: true,
    },
    size: {
      type: 'decimal',
      configurable: false,
      required: true,
    },
    duration: {
      type: 'decimal',
      configurable: false,
      required: false,
    },
    sample_rate: {
      type: 'decimal',
      configurable: false,
      required: false
    },
    url: {
      type: 'string',
      configurable: false,
      required: true,
    },
    ...
  },
  ...
};

```

Then I implemented a simple media info request via `ffprobe` to read out the additional informations and store it in the object fileInfo inside the [upload.js](https://github.com/strapi/strapi/blob/main/packages/core/upload/server/services/upload.js) service:

``` javascript

const ffprobe = require('ffprobe');
const ffprobeStatic = require('ffprobe-static');

...

  async enhanceAndValidateFile(file, fileInfo = {}, metas = {}) {

    console.log(file.path);
    const extendedFileInfo = await ffprobe(file.path, { path: ffprobeStatic.path });
    console.log(extendedFileInfo);
    for (const stream of extendedFileInfo.streams) {
      fileInfo.duration ||= stream?.duration;
      fileInfo.sample_rate ||= stream?.sample_rate;
      fileInfo.height ||= stream?.height;
      fileInfo.width ||= stream?.width;
    }
	...
  }

  ...

```

And added the informations to the entity object, which than get stored in the database:

``` javascript

    const entity = {
      name: usedName,
      alternativeText: fileInfo.alternativeText,
      caption: fileInfo.caption,
      folder: fileInfo.folder,
      folderPath: await fileService.getFolderPath(fileInfo.folder),
      hash: generateFileName(basename),
      ext,
      mime: type,
      size: bytesToKbytes(size),
      width: fileInfo.width,
      height: fileInfo.height,
      duration: fileInfo.duration,
      sample_rate: fileInfo.sample_rate
    };

```

Of course this is very rough implementation, just to test if it works. I only tested it with .mov, .mp3 and .jpeg files. Further testing required. Also I don't know if this is the right place to fetch this informations. If not, I think, you guys will find a better place in the upload logic to make this meta info extraction. 

# Tradeoffs

Perhaps a small impact in "upload performance", although the performance gain in the media library should predominate

# Alternatives

One could download the file from the upload provider, but as mentioned above there are a few advantages to store these infos in the db as well. 

# Unresolved questions

- What are relevant informations to store?
- Can we select which informations we want from a file?
- With what kind of files ffprobe will work?

