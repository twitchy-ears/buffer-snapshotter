# buffer-snapshotter
Minor mode that keeps snapshots of changed versions buffers (visiting files or not) written to disk, and limits those snapshots by number or time

This was written because there are certain kinds of emacs editing situations where you have a buffer that you want essentially auto-saving, but because it's not visiting a file none of the internal mechanisms will cleanly do it.

It runs some timers and when one goes off it will check the `(buffer-chars-modified-tick)` to work out if it needs to save a snapshot or if the buffer remains unchanged, it only snapshots changed buffers with this mode enabled.

Notably this can be used to protect [Emacs Atomic Chrome](https://github.com/alpha22jp/atomic-chrome) buffers.

It's pretty simple and should depend on nothing external to Emacs but it was written and tested against an Emacs 29.1 install so caution is recommended if you're using something else.  It should be ok from some basic use and testing.

Essentially all you need is this:

    (use-package buffer-snapshot)

Then `M-x buffer-snapshotter` in a buffer you feel needs this mode.

If you want it to clean up old snapshot files whenever you start Emacs then
try something like this:

    (use-package buffer-snapshot
        :config
        (buffer-snapshotter-cleanup-directory))

Where `(buffer-snapshotter-cleanup-directory)` goes through the snapshots directory and deletes every file older than `buffer-snapshotter-maximum-age` seconds, this defaults to 259200 so should cleanup all files older than 3 days.

If you want it to automatically start for certain modes use tricks
like this:

    (add-hook 'atomic-chrome-edit-mode-hook
               (lambda () (buffer-snapshotter 1)))

Interesting variables to play around with and read the docstrings of include:

* `buffer-snapshotter-directory` the directory to store snapshots in, defaults to "buffer-snapshots" under the users `user-emacs-directory`
* `buffer-snapshotter-cleanup-method` allows you to pick cleanup method, it defaults to `#'buffer-snapshotter-delete-excess-by-number` which keeps N versions of each files snapshots at most, set to `#'buffer-snapshotter-delete-excess-by-time` if you want to keep snapshots for a specific amount of time, or create your own function.
* `buffer-snapshotter-keep-versions` numbers of versions to keep if keeping by number
* `buffer-snapshotter-keep-time` seconds to keep a snapshot, defaults to 21600 so 6 hours.
* `buffer-snapshotter-frequency` seconds of idle time before checking to see if a snapshot is worth taking, defaults to 30s.
* `buffer-snapshotter-force-frequency` seconds of time before checking to see if a snapshot is worth taking idle or not, defaults to 600s (10 minutes).
* `buffer-snapshotter-namegen-func` should be a `#'function-symbol` to a function used to generate consistent and filesystem safe names for snapshot files, the default replaces characters with their codepoint numbers for everything not matching `[a-zA-Z0-9_-]`
