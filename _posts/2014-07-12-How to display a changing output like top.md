In my earlier post on linux memory management, I showed you how you can get the exact memory usage of your script by using py_mem.py.
watch command examples
This script, without any options will run once and give you the current memory usage of the system.
But what if you wanted real time memory usage?, In other words, if a program which was using up memory did not run when you ran the script py_mem.py, you would not know about it right?

So how do you make a script print an output like the top command?

One way to do it with this script is to use the -w option

ps_mem.py -w 2

watch command examples

This uses a while loop and runs the command in intervals, But again, this changes the whole output, Wouldn't it be better if only the process whose memory usage changed was refreshed ?

This can easily be accomplished with the watch command.

From the watch man page
“watch runs a command repeatedly, displaying its output (the first screenfull). This allows you to watch the program output change over time. By default, the program is run every 2 seconds”

basic syntax

watch commandname


watch sudo py_mem.py

watch command examples
You can change the time interval with the “-n” option.

watch -n 5 sudo py_mem.py

The above example will run this command every 5 seconds instead of the default 2.

Now if you look at the example, sure it is changing, but wouldn't it be much easier to read if the output that changed was highlighted ?, The watch command with the -d option does just this.

watch -d sudo py_mem.py

watch command-examples
Another option, although I don’t prefer it, but good to know is that you can make the highlight sticky i.e. the change is always highlighted by using the "-d=cumlvative" option

watch -d=cumulative sudo py_mem.py

watch command examples

The watch command helped me run this script continuously and troubleshoot memory issues in real time, if you have any suggestions or know any other tricks with watch then do let me know.

You may also like to read how to capture passwords with tcpdump or Monitor Linux File System events with Inotify-tools

Update: For a list of Bash and Linux Tutorials see https://github.com/Leo-G/DevopsWiki
