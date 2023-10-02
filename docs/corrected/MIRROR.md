# How to Add a stal/IX Mirror

List of source mirrors used to build stal/IX:

https://github.com/pg83/ix/blob/main/pkgs/die/scripts/mirrors.txt

You can help the project by hosting such a mirror and making it available to the world!

The structure of the mirror is very simple - it is a single-level list of files, and the name of each file is sha256sum of the data contained in this file.

So, for example, you can copy all the contents of one of the mirrors and make a copy available on the Internet.

Or you can run a ready-made script in cron, for example, once an hour:

```
# ~/ix - clone of https://github.com/stal-ix/ix
# script will need gnu make, git, python3, wget in PATH
# output will be in ~/some/dir

cd ~/ix
git pull
./ix recache ~/some/dir
```

[### Instructions for Ubuntu](https://gist.github.com/pg83/4bdb11a2ca3602d949db26b4b2a66781?permalink_comment_id=4687160#:~:text=Commands%20for%20debian,cron.d/ix_recache)

After populating the cache directory, send a PR to add the new mirror to 
https://github.com/pg83/ix/blob/main/pkgs/die/scripts/mirrors.txt
