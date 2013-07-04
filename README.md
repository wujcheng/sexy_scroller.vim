sexy_scroller.vim - Smooth animation of the cursor and the page whenever they move, with easing.
By joeytwiddle, inspired by Terry Ma's smooth_scroll.vim (although there are many others)

Usually when you scroll the buffer or motion to a different part of the
document, Vim will jump there immediately.  But with SexyScroller, Vim will
scroll to the new position smoothly.  As well as looking pretty, this
provides you with visual feedback about the distance you have travelled.

== Options ==

Instead of specifying the scrolling *speed*, SexyScroller asks you to
specify how *slow* you want scrolling to be.  You can store these options in
your .vimrc once you are happy with them.

    :let g:SexyScroller_CursorTime = 5

Sets the time taken to move the cursor one line (in milliseconds), or
set it to 0 to never scroll the cursor.  However, you may not see the
cursor during animation, in which case you can   :set cursorline

    :let g:SexyScroller_ScrollTime = 10

Sets the time taken to scroll the buffer one line.  (I like to pretend
the buffer is "heavier" than the cursor.)

    :let g:SexyScroller_MaxTime = 500

Sets the maximum time for long scrolls.

    :let g:SexyScroller_EasingStyle = 1

Sets the easing style (how scrolling accelerates and decelerates),
where:

       1 = start fast, finish slowly            (practical)

       2 = start slow, get faster, end slowly   (sexiest)

       3 = constant speed                       (dull)

    :let g:SexyScroller_DetectPendingKeys = 1   /   0

Interrupts the animation if you press a key.  Should resume animation
if they key you pressed also causes scrolling, otherwise just jumps
directly to the destination.  This feature seems to be working ok now.
Resuming animation looks best with EasingStyle 1.

Finally, a command is provided to enable/disable the scrolling:

    :SexyScrollerToggle

For eye candy, set MaxTime to 1200 and EasingStyle to 2.

Power users may prefer to lower MaxTime to 400, and set EasingStyle 1 or 3.



== Issues ==

- The script has trouble detecting a scroll event if the cursor did not move.  This is because we listen for CursorMoved events; Vim does not provide any WindowScrolled event!

  If the script does fail to detect when a scroll occurs, it will eventually notice it later, when the cursor *does* move.  This will trigger an animation later than expected from a point which we already left some time ago.  This looks wrong!

  Since this affects <C-E> and <C-Y>, we remap them to compensate.  However if you have mapped any alternative keybinds or commands which scroll the page without moving the cursor, these will not work.  A workaround is to append <BS><Space> to any such mappings, to move the cursor and move it back again!

  Other keys similarly affected are the various `z` commands under |scroll-cursor|.  They are hard to map.

- We cannot detect the scrolling through the document that occurs during a `/` search when 'incsearch' is enabled, again from a lack of events.  (This does not affect `n` and `N`, which work fine.)  TODO: We could temporarily disable animations when `/` or `?` are initiated and 'incsearch' is enabled (until the next CursorMoved or a CursorHold).  Alternatively, we could attempt to implement a fake `/` interface.

- I have disabled smooth horizontal animation of the cursor because I can never see the cursor when it's moving, even with 'cursorcolumn' enabled!  Scrolling in this case is pointless: it just looks like Vim is briefly frozen!  In fact the cursor is also invisible during vertical scrolling, although 'cursorline' is visible while it is moving, if enabled.  CONSIDER TODO: A workaround for both axes might be to perform the requested movement as a sequence of feedkeys(), rather than calls to winrestview.  We would need to avoid re-triggering ourself on those CursorMoved events, with :noauto or with a flag.

- PageUp, PageDown, Ctrl-U and Ctrl-D do not consistently trigger a getchar(), so DetectPendingKeys does not always work for them.  This may be system-dependent.  Simpler keystrokes like { and } never fail.  The result is that subsequent keypresses will not interrupt animation, but will start a second animation separately after the animation of the earlier keypresses.

- Plugins which use :noauto (TagList for example) will not fire CursorMoved when they actually happen.  If we then focus the window later, this will lead to late detection and an out-of-date animation being performed.

- Resizing the window may cause the topline/leftcol to change without firing a CursorMoved event, with the usual consequences.

- With 'cursorline' enabled, the cursor will animate after a mouse click, which does not look natural.  In this case, it should simply jump without any animation.  I cannot think of any way to fix this.

- Animation does not work well with mouse scrolling.  I can't think of any way to work around this.  If you scroll with the mouse more than the keys, this plugin might start ao annoy you.  But it might be the thing that trains you to improve your Vimming.  :)

- Our ability to scroll smoothly is limited by the presence of long wrapped lines at the top of the window. (For example if line 340 is long, wrapping to 6 screen lines, then since we cannot set 'topline' to "partway through line 340", the displayed text is forced to jump 6 lines when we set 'topline' to 341.)

- Folded lines affect the effort/time-taken calculations.  So it takes MaxTime to scroll 1000 lines out of view, even if those 1000 lines have been folded down and appear visually as one line!

CONSIDER TODO: Make a list of exclude keys, and map them so that they set w:SexyScroller_Ignore_Next_Movement.  For example this could apply to `/` and `?` with 'hlsearch' enabled, and maybe also to `d`.

CONSIDER TODO: We could optionally enable cursorline whilst scrolling.  (Reproducing the functionality of highlight_line_after_jump.vim)

TODO: We should politely store and restore lazyredraw if we are going to continue to clobber it.

