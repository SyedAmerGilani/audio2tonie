# audio2tonie

---

A fast and easy way to transcode any audio files to a format which is playable by a Tonie box. 

This fork is based on the [opus2tonie.py](https://github.com/bailli/opus2tonie) script. But since it's quite difficult to set up and has lots of dependencies (ffmpeg, Python, Google Protobuf, etc.) I decided to build a docker container. Some small updates have been implemented as well as a shell script which is used as a wrapper for the entrypoint.

### Installation

#### Option 1: Use pre-built image from GitHub Packages (Recommended)

The Docker image is automatically built and published to GitHub Packages via CI/CD pipeline:

```bash
docker pull ghcr.io/syedamergilani/audio2tonie:latest
```

#### Option 2: Build from source

- `git clone https://github.com/SyedAmerGilani/audio2tonie.git`
- `cd audio2tonie`
- `docker build -t audio2tonie .`

### Usage

The intention is to run this container on-demand and only as long as the file conversions are running. 

Invoke it from your desired host directory where your audio files are stored. In these examples, the current host directory (`$(pwd)`) will be mounted into the container's `/data` folder (recommended).

**Note:** Replace `ghcr.io/syedamergilani/audio2tonie:latest` with `audio2tonie` if you built the image locally.

```
docker run --rm -v $(pwd):/data ghcr.io/syedamergilani/audio2tonie:latest transcode 
            -s/--source SOURCE 
           [-o/--output OUTPUT]
           [-r/--recursive] 
           
required argument:
  SOURCE            the input source: a single file, a list (*.lst) or a folder

optional arguments:
  OUTPUT            the output filename (default: same as input with .taf extension)
  -r/--recursive    creates a taf file for each subfolder recursively
```

### Examples

- Convert a single file `audiobook.mp3` from your current directory
```
Command:
docker run --rm -v $(pwd):/data ghcr.io/syedamergilani/audio2tonie:latest transcode -s /data/audiobook.mp3

Output: 
audiobook.taf
```
- Convert a single file `audiobook.mp3` from your current directory with given output name
```
Command:
docker run --rm -v $(pwd):/data ghcr.io/syedamergilani/audio2tonie:latest transcode -s /data/audiobook.mp3 -o /data/lullaby.taf

Output: 
lullaby.taf
```

- Convert all files from a given folder into one taf file (with chapters for each file)
```
Command:
docker run --rm -v $(pwd):/data ghcr.io/syedamergilani/audio2tonie:latest transcode -s /data/sandmann

Output:
sandmann.taf (with chapters)
```
- Convert all files from a list `MyFavoriteList.lst` (one file per line) into a single taf (with chapters for each line/file)
```
MyFavoriteList.lst content:
audio.mp3
music.m4a
news.opus
trailer.mp3

Command:
docker run --rm -v $(pwd):/data ghcr.io/syedamergilani/audio2tonie:latest transcode -s /data/MyFavoriteList.lst

Output: 
MyFavoriteList.taf (with 4 chapters)
```
- Convert all subfolders in a given folder into one taf per subfolder (with chapters for each file)
```
Subfolders in current directory: 
 |-Episode 01
 |-Episode 02
 |-Episode 03

Command:
docker run --rm -v $(pwd):/data ghcr.io/syedamergilani/audio2tonie:latest transcode -s /data -r

Result: Episode 01.taf, Episode 02.taf, Episode 03.taf
```

### Chapters

When using a list or folder(s) as source, chapters will be created automatically.

### CI/CD Pipeline

This project includes a GitHub Actions workflow that automatically:
- Builds the Docker image for multiple architectures (linux/amd64, linux/arm64)
- Pushes the image to GitHub Container Registry (ghcr.io)
- Tags images based on:
  - Branch names for feature branches
  - `latest` tag for the main branch
  - Semantic version tags (e.g., `v1.0.0`) when creating releases
- Generates build attestations for supply chain security

The workflow is triggered on:
- Pushes to `main`, `develop`, and `feature/*` branches
- Pull requests to `main` and `develop` branches
- Tag pushes (for releases)

### Some useful resources
* Tonie audio file format: https://github.com/toniebox-reverse-engineering/toniebox/wiki/Audio-file-format
* Source of this fork: https://github.com/bailli/opus2tonie
