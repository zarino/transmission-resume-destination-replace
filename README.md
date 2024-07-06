# transmission-resume-destination-replace

It’s a pain to move more than a handful of seeding files in [Transmission](https://transmissionbt.com/), since you have to do them one by one, selecting “Set location…” from the right-click menu in the UI.

If you just move the actual file, without telling Transmission, it’ll complain that the file has disappeared.

Multiply this by a couple of hundred (because, say, you’ve renamed the directory that all of your completed torrents are seeded from) and you’ve got a problem.

It turns out, Transmission stores each seeding file’s expected path (`destination`) in a `.resume` file, one per seed, inside its config directory.

So, no problem, right? Once you’ve moved your seeding files, you just find-and-replace the old path with the new one, across all these `.resume` files, and Transmission will know where to find the seeds?

Nope, there’s one final wrinkle.

Transmission’s `.resume` file format isn’t just a JSON file or something simple. It uses a special nested key-value serialisation format a bit like [Netstring](https://en.wikipedia.org/wiki/Netstring) or the [PHP serialization format](https://en.wikipedia.org/wiki/PHP_serialization_format), where values are preceded by their length, and a colon. So, if you just blindly string-replace the path, the length prefix won’t match, Transmission won’t be able to parse the `.resume` file, and it’ll ignore it and start again.

The part of a `.resume` file that we care about looks something like this…

```
…11:destination27:/downloads/completed/videos3:dnd…
```

Note the “destination” (the path that I’ve previously “Set location…” of this torrent to) is `/downloads/completed/videos` and that is a string 27 characters long, so it’s prefixed by `27:`.

If I were to string-replace that file, swapping `/downloads/completed` with `/downloads/complete` (which is one character shorter), the new length of the “destination“ path `/downloads/complete/videos` (26 characters) wouldn’t match the pre-existing prefix (`27:`) so parsing would fail.

What you need to do is replace the path _and_ update the length prefix to match the length of the new (complete) destination path.

That’s what this Python script does.

## How to use it

Pause all your torrents in Transmission, then run the script like this:

```
trdr --search "/downloads/completed" --replace "/downloads/complete" transmission/resume/*.resume
```

Move the files to the new location (so Transmission finds them where it now expects them), re-start your torrents, and you’re done!

## Run the tests

```
script/test
```
