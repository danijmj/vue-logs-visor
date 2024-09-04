# Logs visor in Vue.js
Logs visor in Vue.js 3 with a custom drag&drop.

Use the vue-cli-service and pnpm like package manager.

The logs show if the image url has content or if the file is a pdf file with a valid pdf, with a basic necessary information.

## The log format
The log file needs to be a json file with the next format:
[
    {
        "type":number, // 1 if is a image file, 0 if is a pdf
        "url":String, // Url of the resource
        "finded":bool, // true if the file has valid content
        "titn":"String" // A important data of the log
    }
]


## Project setup
```
pnpm install
```

### Compiles and hot-reloads for development
```
pnpm run serve
```

### Compiles and minifies for production
```
pnpm run build
```

### Lints and fixes files
```
pnpm run lint
```

### Customize configuration
See [Configuration Reference](https://cli.vuejs.org/config/).
