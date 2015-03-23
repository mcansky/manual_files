# Tmux

When you connect to a remote server you are quickly tempted to have another console in parallel so you can do two things at the same time. You could be looking at a man page on one, and doing an action on the other. Or you could be monitoring some network traffic on one while changing some configuration on the other.

And then you want a third one, and a forth etc ...

You could also be using a very unstable or unreliable connection to the remote host and you don't want to loose what ever has been going on so far.

Those 2 things (multiple sessions inside a session, persistance of session) are the first features one is introduced to when discovering tmux or its older cousin screen. Both have a similar set of features but the former has been picking up speed in the last few years.

In short Tmux and Screen can be described as "Command line window managers".

## Creating a session

Tmux is very simple to start : just call ```tmux``` in a term and a session will be created.o

You can then start your work.

## Opening up more tabs

Then you might want to have another console with more to do in there. Easy : Ctrl-b-c and a new tab is created. You can start to work in there and the other tab / console is still going.

## Switching between tabs

To verify that you can switch between tabs using different short cuts. The base one is Ctrl-b-n (next). You will switch from one tab to the next in a loop.

You can also go to the previous one (if you have many of them than ca be useful) with Ctrl-b-p (previous).

## Sharing a tmux

All this is fun but one interesting thing to do is to share a tmux session with someone else. You see, it's like sharing screens.
If you and someone else want to do that you just need to both attach the session from the same user. So if you have a work host somewhere with you and a friend sharing a user you can both attach a tmux session.

Once that is done you both see the same active tab and each key press is seen by both of you. That is a great way to do pair programming for example. If you pair this with a voice call for example it turns a remote work environment into a far more productive pair programming setup.

## Epilogue

tmux is a tool to have on every host, most of the time it's part of my default bootstrap scripts on servers and saves me time quite often.

Bad internet connections are the most common problem you can solve with it but a remote pairing session is also a good use of it.

