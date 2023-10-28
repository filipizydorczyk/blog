This repository is blog that you can read on [https://www.filipizydorczyk.pl/articles](https://www.filipizydorczyk.pl/articles). This website is using my content managment system that I wrote with [Nest](https://github.com/nestjs/nest). You can find it in one of my repos called [personal-cms](https://github.com/filipizydorczyk/personal-cms).

This CMS is headless and file based which means that has no UI and all my posts are written with markdown files. When I am done writing a post I push it with the commit to this report. Than my CMS will pull from this repo and provide all articles via REST API which my website is using. This CMS is very simple and written just for me so it only has featured I need so I wouldnt recommend using that.

There are 2 direcotories:
 - `media` - this is where all photos or videos should be stored. If you want somthing more efficient you can host some CDN server and just provide a link to a photo. At the end of the day its all markdown links anyway.
 - `articles` - this is where I write my actual posts. Each posts is required to have `.md` file and name of this file will be the name of the article used to fetch it via API. If you want to add some metadata to article that will be accessible via api you can also create `.json` file with same name. The json you will provide here will be available in API response so you can modele it any way you want as long as you are consisten.

 To look for mistakes I use [languagetool](https://languagetool.org/pl)