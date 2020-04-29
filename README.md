# Shouts Into the Void

## Mini Lecture on the GameBoy Advance forthcoming:
Student comes to office hours with his space game -- randomly here and there we see brown boxes for his sprites instead of the expected asteroids (they look great most of the time)...

I go through my regular debugger tools for the [GBA emulator](https://github.com/visualboyadvance-m/visualboyadvance-m) we use and everything looks good, so I'm stumped and have him send me the code so I can look into it more and move on to help another student.

Fast-forward a few hours and I finally know what's going on, but it took me an hour to find the _single bit_ he was accidentally setting.

The problem:
![Bad Bit](./img/bad_bit.png)
(not my art, used with permission)

The GBA sprites use 4 shorts in memory to control all of their attributes (row, column, palette settings, flipping, size, shape, etc.). It turns out he's accidentally switched row and column (attribute0 is for the row and attribute1 is for the column).

Since the screen is landscape 3:2 aspect ratio, the GBA uses one more bit for the column location than is does for the row location. We provide some helpful macros in a library we build up over the course of our labs to mask ints correctly to fit into the available bits (8 for row and 9 for column).

Since he'd flipped row and column, he'd flipped the masks as well, allowing him to set a bit in attribute 0 that wasn't a location bit at all... Instead it was a [display mode bit](https://www.coranac.com/tonc/text/regobj.htm#sec-oam-entry) that setting to 1 turns the sprite into an _affine sprite_. Since he wasn't setting any affine matrices ([other OAM entry fun with 8.8 fixed point encoding and a little linear algebra](https://www.coranac.com/tonc/text/affine.htm)), he was getting an affine matrix of 4 zeros, which the GBA hardware interprets as "zoom in all the way to the center pixel of the sprite and display only that single color"...

It took me an hour to finally dig into the memory viewer and notice the rogue bit! I should have read his code more closely :sweat_smile: . I thought it was interesting enough to share. I might just be socially starved :smile:. I just emailed the student with the mini-lecture explanation, but told the student to just switch the colmask and rowmask since the game works well already. Every little bit matters!

Fixed version:
![Good Bit](./img/good_bit.png)
(not my art, used with permission)

Cheers and goodnight everyone.

2020-04-28T10:32:07-0400

## Massive overhaul today
So, I made my wife spend time with me a few nights ago updating her professional site (hosted in a very similar manner to this one) and it made me feel like I should actually spend a little time on my own pseudo-site as well (this very one!). I opened up the repo and regretfully remembered why I don't work on it very much -- it was written in raw HTML as a single file and it looks fairly terrible. I use [Jekyll](https://jekyllrb.com/) themes and Markdown when working on my wife's site, since it's much easier for her to work with (though she only has once or twice -- sigh), so I decided I too should benefit from this architecture.

I chose a theme (["Dinky"](https://pages-themes.github.io/dinky/)) and started hacking. I wanted to manage my site as a series of posts, but I still haven't sussed that out yet. So for now, this site has switched to a single massive Markdown file and it looks slightly less awful. Working with it on WSL has also been a bit of a mixed bag, since Jekyll only notices my edits some of the time, but it did at least warn me that might be the case:

```
Auto-regeneration may not work on some Windows versions.
Please see: https://github.com/Microsoft/BashOnWindows/issues/216
If it does not work, please upgrade Bash on Windows or run Jekyll with --no-watch.
```

I'll come back to it soon, hopefully, since I still feel the need to keep up with this thing the way I'm always insisting my wife should do with hers.

2019-11-13T09:47:33-0500

## AWS CLI (but really python) woes
I'm currently working on an application with lots of foreign users who (unsurprisingly) like to upload documents with unicode characters in their names. Today, we're alerted to a file which can't be opened because its "missing" according Working on an application with lots of foreign users who (unsurprisingly) like to upload documents with unicode characters in their names. Today, we're alerted to a file which can't be opened because it's "missing" according to the app.

I start digging around s3 using the awscli python package and grepping around the results from `aws s3 ls --recursive`. I don't see the file they're looking for, but I do see a very similarly named one when I loosen my grep command. All the unicode characters have been replaced by question marks in the file name! Okay, somehow we're dealing with a bad interaction with S3, beause, surely, S3 can handle unicode filenames. I start poking around the ruby gem we're using to do these uploads and despite a few red herrings, it turns out its not rewriting URLs, so it must be a problem with how our app is using it.

I dig around and determine we're also not doing such a modification to the URL! Meanwhile, I scan our entire S3 bucket looking for file names with question makrs in their names and find hundreds! What gives?!?

The culprit
I was doing all this searching around S3 using the awscli python package and piping the results into grep (and sometimes piping into wc, etc.). But then I noticed, when I did just the `aws s3 ls` command, the results in the terminal *did* have the expected unicode characters! The files were there and perfectly fine in S3 all along. So is grep the culprit in the question marks? The man page for the version of grep I'm using on the El Capitan Mac have this interesting line:

| The grep utility does not normalize Unicode input

So I brew install ggrep and start using it instead -- but see the same issue... On a lark, I pipe the output from awscli into `cat` and voila, still question marks! Piping the output from this python cli executable is leading to different output than allowing the results to go directly to the terminal. I start doing a little Googling and find that python programs have issues with checking the default system encoding (because mine is definitely set to UTF-8, but python says its set to ascii), but Stack Overflow yielded this gem.

`export PYTHONIOENCODING=utf8` has now been added to my .bash_profile to avoid this silly problem when using the awscli gem in the future. Back to sane piping without losing unicode characters!

2016-09-15T14:59:38-0400

## And while trying to deploy this very app,
I discovered, Heroku's dyno died on me at least 3 weeks ago. Only a `heroku restart` from the CLI tool brought the app back to life. Very unusual/annoying --- I suppose I might have to move to something more reliable at some point. The irony is, that something will probably be GitHub...

2016-04-12T16:04:56-0400

## Meanwhile,
GitHub's ["secret" whitespace-ignoring feature](https://github.com/blog/967-github-secrets) is apparently not working and it's slowing me down today.

2016-04-12T15:41:20-0400
