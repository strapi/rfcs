- Start Date: 2020-03-29
- RFC PR: (leave this empty)

# Summary

Introduce folders structure to `strapi-plugin-upload` and upload modal in `strapi-plugin-content-manager`

# Motivation

Right now it's hard to organize your assets. It works fine when you have few of them, but when the amount of media content items comes to 100 and more, it's getting difficult to work with it.

In case of folders structure, we transfer the task of files management and organization to user and he can decide how it's better to organize his assets specifically for his project.

# Detailed design

## Plugin core

### Virtualized structure

- Folders are not created physically, rather just files are linked to folders entities, that we save in DB.

- All the files are uploaded inside the `upload` folder without any structure. This way doesn't require changes inside upload providers, because folders are virtual.

  ```shell
  Strapi project root folder
  └── /public/upload
      ├── large_picture
      |   └── large_picture_48234s.jpg
      ├── medium_picture
      |   └── medium_picture_48274s.jpg
      ├── small_picture
      |   └── small_picture_a82sas.jpg
      ├── movie.mp4
      └── */*.*
  ```

### **Folder** Model

- We will need to add a new model `Folder`.
- Both `File` and `Folder` models should be able to be related to Folder.

`Folder.settings.json`

```json
{
  "info": {
    "name": "folder",
    "description": ""
  },
  "options": {
    "timestamps": true
  },
  "attributes": {
    "name": {
      "type": "string",
      "configurable": false,
      "required": true
    },
    "parent": {
      "model": "folder",
      "required": false,
      "configurable": false
    }
  }
}
```

`File.settings.json`

```diff
...
+   "parent": {
+       "model": "folder",
+       "required": false,
+       "configurable": false
+    }
...
```

### New controllers

We will need to add new controllers to work with folders structure:

- Folders

  - createFolder
  - updateFolderInfo
  - deleteFolder
  - moveFolder

- Files
  - moveFile

### Add graphql schema

We will need to add graphql schema resolutions:

- Folders

  - createFodler
  - updateFolderInfo
  - deleteFolder
  - moveFolder

- File
  - moveFile

**deleteFolder** should recursively delete everything inside of it.

## Plugin Admin

> I really want the UI to be reworked, to make it more user-friendly and to improve usability. I like the way they work with folders and files in Google Drive: Drag & Drop upload, moving file/folder by just dragging it on the folder, selecting range of files using `Ctrl` or `Shift` + `Click`

Main scopes to rework:

- Plugin page of `strapi-plugin-upload`

  - Add folder
  - Move folder
  - Rename folder
  - Delete folder

  > I don't think that majority of filtering filtering options that are done right now not useful. Most wanted features is search, recent uploads / recently used files and sorting by name (ASC/DESC). It should be a question to strapi community, what do they feel useful.

- Modal to select image or upload to a specific directory in `strapi-plugin-content-manager`

# Tradeoffs

- Implementing this proposal means rebuilding `strapi-plugin-upload` and upload modal in `strapi-plugin-content-manager`

  **Backend**

  - Implementing new services, controllers, Folders model, grapql schema methods
  - Rewriting already existing services of File, adding move method in graphql
  - Changing File model

  **Admin UI**

  - Rebuilding UI of upload plugin to work with folders
  - Rebuilding upload modal

- According to what I described above, it should not affect other currently implemented features. Models changes should not break anything, so no migration scripts and guides are required.

- Implementing this proporsal also means reworking documentation and some tutorials to describe work with folders.
