---
layout: post
title:  "Avoid Problems With Sample Clearence"
date:   2024-05-02 13:59:30 -0400
published: true
---

## Problem With Sample Clearence
Nowadays it's pretty rare to find music that does not contain a sample. Everyone seems to be sampling, be it Drake, Nick Mira, Kendrick, and a bunch of other musicians. Sampling is powerful in that it provides inspiration and direction when making music. This idea essentially revolves around repurposing. As a music producer, the problem I personally have is keeping track of the places where I sample from. At the end of the day you can't simply release a song with a sample in it without clearing it first. This might not be a big problem for old heads that still use vinyl but it's definitely a problem when your sampling from tiktok, and downloading it through some website that sometimes does not transfer metadata.

## My Solution
The simple way to get around this issue is through scripting. Below is a simple script using python that will currently download from Spotify, Instagram, and Youtube. It has an easy to use CLI and with the library I used (click) it's easily customizable to meet your needs. It has a bunch of dependencies which I tried to avoid, but I do not have time to make everything from scratch, also I am not fluent in python.

Tt is far from perfect and legit does not have any error handling whatsoever, but it get's the job done.

## Depedencies
- instaloader
- re
- sanitize_filename
- spotdl
- yt_dlp
- click

## Code
Firstly the folder structure is as follows
```
media_downloader/
├─ utils/
│  ├─ insta.py
│  ├─ spotify.py
│  ├─ yt.py
├─ constants.py
├─ main.py
├─ setup.py

```

## insta.py
```
import instaloader
import re
import constants
import os
from pathlib import WindowsPath

def insta_proc(url, path):
  insta = instaloader.Instaloader()
  
  url_shortcode = re.findall(r'^(?:https?://)?(?:www\.)?(?:instagram\.com(?:/\w+)?/p/)([\w-]+)(?:/)?(\?.*)?$', url)[0][0] # grabbed from here https://stackoverflow.com/a/57043055/20809108
  # the above regex might satisfy more than one sub string, the first one will always be the short code according to the instagram standards
  
  post = instaloader.Post.from_shortcode(insta.context, url_shortcode)
  
  dest = WindowsPath(os.path.join(path, constants.INSTA_IMG_DUMP))
  
  insta.download_post(post, target=dest) 
  
  
```

## spotify.py
```
import json
from sanitize_filename import sanitize
from spotdl import Spotdl
import os 
from dotenv import load_dotenv

load_dotenv()

def spotify_proc(url, file_path):
  
  obj = Spotdl(client_id=os.environ["SPOTIFY_CLIENT_ID"], client_secret=os.environ["SPOTIFY_CLIENT_SECRET"], no_cache=True)
  
  song_objs = obj.search([url])
  
  result = obj.download(song_objs[0]) # returns song object
  
  title = sanitize(result[0].name)    

  # create directory names after the title in metadata
  dir_path = os.path.join(file_path, title)
  
  os.mkdir(dir_path)

  # move default download location to it's respective directory
  default_location = os.path.join(result[1].cwd(), result[1])
  
  new_location = os.path.join(dir_path, result[1])
  
  os.replace(default_location, new_location)
  
  # now write the metadata into file
  jsonf_path = os.path.join(dir_path, f"{title}.json")
  with open(jsonf_path, "w") as f:
      f.write(json.dumps(result[0].json))
```

## yt.py
```
import yt_dlp
import json
import os
from sanitize_filename import sanitize

def yt_proc(url, file_path):
    
    # save metadata
    with yt_dlp.YoutubeDL() as ydl:
        info = ydl.extract_info(url, download=False) 
        
        metadata = json.loads(json.dumps(ydl.sanitize_info(info)))
        
        title = sanitize(metadata["title"])        
    
    # create directory names after the title in metadata
    dir_path = os.path.join(file_path, title)
    
    os.mkdir(dir_path)
    
    # download audio
    ydl_opts = {
        'format': 'wav/bestaudio/best', # this has the list of format available https://github.com/yt-dlp/yt-dlp?tab=readme-ov-file#format-selection
        # ℹ️ See help(yt_dlp.postprocessor) for a list of available Postprocessors and their arguments
        'postprocessors': [{  # Extract audio using ffmpeg
            'key': 'FFmpegExtractAudio',
            'preferredcodec': 'wav',
        }],
        'paths': {
            'home': dir_path
        }
    }   
    
    with yt_dlp.YoutubeDL(ydl_opts) as ydl:
        error_code = ydl.download(url)
        
        jsonf_path = os.path.join(dir_path, f"{title}.json")
        
        with open(jsonf_path, "w") as f:
            f.write(json.dumps(metadata))
```
## constants.py - really redundant for now
```
# for instagram photos
INSTA_IMG_DUMP = "IMG_DUMP"
```

## main.py
```
import click
from utils.yt import yt_proc
from utils.spotify import spotify_proc
from utils.insta import insta_proc
import os

@click.group()
def cli():
  pass

# Youtube sub command
@cli.command()
@click.option('--url', default="", help="enter youtube url")
@click.option('--path', default=os.getcwd(), type=click.Path(exists=True), help="enter download destination")
def yt(url, path):
  yt_proc(url, path)
  
  
# Spotify sub command
@cli.command()
@click.option('--url', default="", help="enter youtube url")
@click.option('--path', default=os.getcwd(), type=click.Path(exists=True), help="enter download destination")
def spotify(url, path):
  spotify_proc(url, path)
  
  
# Instagram sub command
@cli.command()
@click.option('--url', default="", help="enter youtube url")
@click.option('--path', default=os.getcwd(), type=click.Path(exists=True), help="enter download destination")
def insta(url, path):
  insta_proc(url, path)

```


## setup.py
```
from setuptools import setup, find_packages

with open("README.md", "r", encoding="utf-8") as fh:
    long_description = fh.read()
# with open("requirements.txt", "r", encoding="utf-8") as fh:
#     requirements = fh.read()
setup(
    name = 'media-downloader-v1',
    version = '0.0.1',
    author = 'MiguelBBeats',
    author_email = 'leo.miguel.bantl@gmail.com',
    license = 'n/a',  # TODO update 
    description = 'simple cli that combines multiple tools in one, making sampling more efficient', 
    long_description = long_description, 
    long_description_content_type = "text/markdown",
    url = '<github url where the tool code will remain>', # TODO update
    py_modules = ['main'], # list all modules here
    packages = find_packages(),
    # install_requires = [requirements],
    python_requires='>=3.7',
    classifiers=[
        "Programming Language :: Python :: 3.8",
        "Operating System :: OS Independent",
    ],
    entry_points = '''
        [console_scripts]
        miguel=main:cli  
    ''' # when someone enters miguel in console, run cli function inside the main.py module
)
```

