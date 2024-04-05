# Architecture Repository

A POC repository for organizing code-based architecture diagrams and flows

## Running

To run, make sure you have the most recent version of d2tosite. The included binary here is for Linux so Netlify doesn't have to build it. If you are on another platform, you will likely need to install d2tosite locally with your own architecture until the other tool implements automated builds for platform releases.

To run, use the included config file. This will generate output in the `./build` directory, which you can then use something like `serve` to server from the filesystem:

```bash
./d2tosite --config=config.yaml
```

Then make the build directory available:

```bash
cd ./build
serve .
```

Or a nice one-liner:

```bash
./d2tosite --config=config.yaml && serve ./build
```

You can also use `d2 --watch {file_name}` when writing it to watch the results in real time.

**Note:** The config file is set to clean the build directory, so it will be removed from the filesystem prior to running. If you don't want this, set `clean` to `false` in the config file.

## Installing

```bash
curl -fsSL https://d2lang.com/install.sh | sh -s --
```

### Terrastruct Repo

[Terrastruct](https://github.com/terrastruct/d2#install)
