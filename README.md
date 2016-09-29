# gulp-awesome-electron

By using `gulp-awesome-electron` you can build cross platform desktop apps using GitHub Electron straight from your gulp build process. This module is a fork with some small extensions from `gulp-atom-electron`.

##Enhancements compared to `gulp-atom-electron`

 - include icons for Windows executables when build on MacOS or Linux
 - provide custom `path` to cache electron binaries

### Installation

```bash
$ npm i gulp-awesome-electron --save-dev
```

### Building for Windows from MacOS or Linux

If you want to build for windows either from a MacOS or Linux, you've to install [Wine](https://www.winehq.org/) version `^1.6.0`.

### Usage

You can use `gulp-awesome-electron` for tow different tasks:

 - for **packaging** your application
 - for **downloading** an electron release to your disk

#### How to Package Your Application

You should source your app's files using `gulp.src` and pipe them through
`gulp-atom-electron`. The following task will create your application in
the `app` folder, ready for launch.

```javascript
const gulp = require('gulp'),
    symdest = require('gulp-symdest'),
    electron = require('gulp-awesome-electron');

gulp.task('default',() => {
	return gulp.src('src/**')
		.pipe(electron({
		    version: '1.4.0',
            platform: 'darwin',
		    cache: '~/.electron-cache/',
		    companyName: 'your company name here',
		    token: process.env.GH_TOKEN,
		    linuxExecutableName: 'AwesomeElectron'
        }))
		.pipe(symdest('app'));
});
```

**Note:** It is important to use `gulp-symdest` only because of the OS X
platform. An application bundle has symlinks within and if you use `gulp.dest`
to pipe the built app to disk, those will be missing. `symdest` will make
sure symlinks are taken into account.

Finally, you can always pipe it to a **zip archive** for easy distribution.
[joaomoreno/gulp-vinyl-zip](https://github.com/joaomoreno/gulp-vinyl-zip) is recommended:

```javascript
const gulp = require('gulp'),
    zip = require('gulp-vinyl-zip'),
    electron = require('gulp-awesome-electron');

gulp.task('default', () => {
	return gulp.src('src/**')
		.pipe(electron({
		    version: '1.4.0',
            platform: 'darwin',
            cache: '~/.electron-cache/',
            companyName: 'your company name here',
            token: process.env.GH_TOKEN,
            linuxExecutableName: 'AwesomeElectron'
        }))
		.pipe(zip.dest('app-darwin.zip'));
});
```

#### How to Download Electron

There's also a very handy export `electron.dest()` function that
makes sure you always have the exact version of Electron in a directory:

```bash
$ npm i gulp run-sequence gulp-awesome-electron --save-dev
```

```javascript
const gulp = require('gulp'),
    runSequence = require('run-sequence'),
    electron = require('gulp-awesome-electron');

let download = (version, platform) => {
    return electron
        .dest('~/.electron-cache', {
            version: version,
            platform: platform
       });
};

gulp.task('download-windows', () => {
    return download('1.4.0', 'win32')
});

gulp.task('download-linux', () => {
    return download('1.4.0', 'linux')
});

gulp.task('download-macos', () => {
    return download('1.4.0', 'darwin')
});

gulp.task('default', (done) => {
    return runSequence(['download-windows', 'download-linux', 'download-macos'], done);
});
```

This will place a vanilla Electron build into the `electron-build` directory.
If you run it consecutively and it detects that the version in the destination directory
is the intended one, it will end up in a no-op. Else it will download the provided version
and replace it.


### Options

You **must** provide the following options:
- `version` - the [Electron version](https://github.com/atom/electron/releases) to use
- `platform` - kind of OS (`darwin`, `linux` or `win32`)

The following options are **optional**:
- `quiet` - suppress a progress bar when downloading
- `cache` - provide a custom folder path where electron binaries will be cached. (Defaults to `os.tempDir/gulp-awesome-electron-cache`)
- `token` - GitHub access token(to avoid **request limit**. You can grab it [here](https://github.com/settings/tokens))
- `arch` - the processor architecture (`ia32`, `x64`)

- **Windows only**
	- `winIcon` - path to an `.ico` file
	- `companyName` - company name
	- `copyright` - copyright statement

- **MacOS (darwin) only**
	- `darwinIcon` - path to an `.icns` file
	- `darwinBundleDocumentTypes` - ([reference](https://developer.apple.com/library/ios/documentation/filemanagement/conceptual/documentinteraction_topicsforios/Articles/RegisteringtheFileTypesYourAppSupports.html)) array of dictionaries, each containing the following structure:
	 - `name` - the `CFBundleTypeName` value
	 - `role` - the `CFBundleTypeRole` value
	 - `ostypes` - the `CFBundleTypeOSTypes` value, a `string` array
	 - `extensions` - the `CFBundleTypeExtensions` value, a `string` array of file extensions
	 - `iconFile` - the `CFBundleTypeIconFile` value

- **Linux only**
	- `linuxExecutableName` - overwrite the name of the executable in Linux
