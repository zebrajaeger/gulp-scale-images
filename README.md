# gulp-scale-images

**[Gulp](https://gulpjs.com) plugin to make each image smaller. Combined with [`flat-map`](https://npmjs.com/package/flat-map), you can create multiple variantes per image**, which is useful for [responsive images](https://developer.mozilla.org/en-US/docs/Learn/HTML/Multimedia_and_embedding/Responsive_images).

![ISC-licensed](https://img.shields.io/github/license/derhuerst/gulp-scale-images.svg)

## Motivation

This is 'heavily inspired' by gulp-scale-images (https://www.npmjs.com/package/gulp-scale-images). And the lib does a great job.

So why fork this?

On my Orange and Raspberry Pi, the Sharp-Code node crashes because of memory errors and i did not found a workaround.
So i decided to take the plugin and separate the parts (Gulp plugin and resizer).

Now there is a unique interface and you can decide to use sharp or jimp as resizing engine.  

## Installing

```shell
npm install @zebrajaeger/gulp-scale-images @zebrajaeger/gulp-scale-images-resize-sharp --save-dev
```

or 

```shell
npm install @zebrajaeger/gulp-scale-images @zebrajaeger/gulp-scale-images-resize-jimp --save-dev
```

## Usage

`gulp-scale-images` expects the instructions for each file to be in `file.scale`. They may look like this:

```js
{
	maxWidth: 300, // optional maximum width, respecting the aspect ratio
	maxHeight: 400, // optional maximum height, respecting the aspect ratio
	format: 'jpeg', // optional, one of ('jpeg', 'png', 'webp')
	quality: 80, // optional and only for jpeg target format
	withoutEnlargement: false // optional, default is true
}
```

*Note*: You must specify at least one of `maxWidth` and `maxHeight`.

An example, we're going to generate *two* smaller variants for each input file. We're going to use [`flat-map`](https://npmjs.com/package/flat-map) for this:

```js
const gulp = require('gulp')
const flatMap = require('flat-map').default
const scaleImages = require('@zebrajaeger/gulp-scale-images')

// choose one of them
const scaleImagesResize = require('@zebrajaeger/gulp-scale-images-resize-sharp')
const scaleImagesResize = require('@zebrajaeger/gulp-scale-images-resize-jimp')

const twoVariantsPerFile = (file, cb) => {
	const pngFile = file.clone()
	pngFile.scale = {maxWidth: 500, maxHeight: 500, format: 'png'}
	const jpegFile = file.clone()
	jpegFile.scale = {maxWidth: 700, format: 'jpeg'}
	cb(null, [pngFile, jpegFile])
}

gulp.src('src/*.{jpeg,jpg,png,gif}')
.pipe(flatMap(twoVariantsPerFile))
.pipe(scaleImages(scaleImagesResize))
.pipe(gulp.dest(…))
```

### Definining scale instructions based on metadata

You can let `gulp-scale-images` read the image metadata first, to device what to do with the file:

```js
const readMetadata = require('gulp-scale-images/read-metadata')
const through = require('through2')
const scaleImages = require('@zebrajaeger/gulp-scale-images')

// choose one of them
const scaleImagesResize = require('@zebrajaeger/gulp-scale-images-resize-sharp')
const scaleImagesResize = require('@zebrajaeger/gulp-scale-images-resize-jimp')

const computeScaleInstructions = (file, _, cb) => {
	readMetadata(file.path, (err, meta) => {
		if (err) return cb(err)
		file.scale = {
			maxWidth: Math.floor(meta.width / 2),
			maxHeight: Math.floor(meta.height / 2)
		}
		cb(null, file)
	})
}

gulp.src(…)
.pipe(through.obj(computeScaleInstructions))
.pipe(scaleImages(scaleImagesResize))
.pipe(gulp.dest(…))
```

### Custom output file names

By default, `gulp-scale-images` will use `{basename}.{maxWidth}w-{maxHeight}h.{format}` (e.g. `foo.500w-300h.jpeg`). You can define a custom logic though:

```js
const path = require('path')
const scaleImages = require('@zebrajaeger/gulp-scale-images')

// choose one of them
const scaleImagesResize = require('@zebrajaeger/gulp-scale-images-resize-sharp')
const scaleImagesResize = require('@zebrajaeger/gulp-scale-images-resize-jimp')

const computeFileName = (output, scale, cb) => {
	const fileName = [
		path.basename(output.path, output.extname), // strip extension
		scale.maxWidth + 'w',
		scale.format || output.extname
	].join('.')
	cb(null, fileName)
}

gulp.src(…)
.pipe(through.obj(computeScaleInstructions))
.pipe(scaleImages(scaleImagesResize, computeFileName)) // not that we pass computeFileName here
.pipe(gulp.dest(…))
```

### `gulp-scale-images` works well with

- [`flat-map`](https://www.npmjs.com/package/flat-map) – A flat map implementation for node streams. (One chunk in, `n` chunks out.)
- [`replace-ext`](https://www.npmjs.com/package/replace-ext) – Replaces a file extension with another one.


## Contributing

If you have a question or have difficulties using `gulp-scale-images`, please double-check your code and setup first. If you think you have found a bug or want to propose a feature, refer to [the issues page](https://github.com/zebrajaeger/gulp-scale-images/issues).
