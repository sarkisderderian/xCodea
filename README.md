# xCodea

A live coding environment for large Codea projects.

### Why

xCodea lets you seamlessly switch working on your Codea project from your iPad to your laptop/desktop (and back), to take advantage of

- a proper keyboard! :)
- extended screen estate thanks to your 4 monitors
- your favourite text editor/IDE, with powerful features such as
	- autocompletion
	- refactoring
	- advanced code navigation with fast keyboard shortcuts
- inline or external documentation always visible
- being able to easily find and integrate code snippets (or whole libraries) from the internet without interrupting your workflow
- git (or your favourite SCM) for versioning, experimentation, and team collaboration, integrated into your workflow
- powerful tools for managing large projects with lots of files and dependencies
- ...

And most importantly:

- your project _runs live_ (on your iPad) - you see the effect of changes in the code in (almost) realtime
- your project _runs in a sandbox_   - whenever you make a mistake only that part of the code is affected, while the rest of the project keeps running; fix the error and you're right back in the program flow without restarting anything


### Setup
Assuming your computer's IP is 192.168.1.10, you will keep your projects in a folder named CodeaProjects (this folder is generally referred to as _projectsRoot_) and you'll work on a Codea project called MyCoolGame:

- `git clone http://github.com/lowne/xCodea.git CodeaProjects; cd CodeaProjects` - or [download](https://github.com/lowne/xCodea/archive/dev.zip) the ZIP and extract its contents into CodeaProjects
- `nano xCodea/EDIT_THIS.lua` - or open EDIT_THIS.lua inside the xCodea subfolder with a text editor
- put in the (static) IP address or hostname (spaces and apostrophes will likely cause trouble, though) of your computer, as in `xCodea_server = "http://192.168.1.10:49374"` then save
- start the server: `./xcodeaserver.py MyCoolGame` (alas this must necessarily be done in a terminal for now)
- on the iPad, point Safari to the server: type `http://192.168.1.10:49374` in the location bar
- select the entire text and copy it
- launch Codea, long-press the _Add New Project_ button then tap _Paste into project_; call it **xCodea**
- tap the right-pointing triangle to run xCodea :p

### Running the server

[ **TODO** command line options etc]

### Connection and syncing

When xCodea on the iPad connects to the xCodea server any changes (files or dependencies that were updated, added or removed from either side) are synced. 
In practice almost always you'll have been working on your project on the iPad while away from the computer. When you connect to xCodea it'll sync these changes back to the computer.
In case of changes from both sides _on the same file_, the iPad version wins by default, overwriting the file on your computer.

- In special circumstances (such as having two iPads, or calamitous corruption on your computer) you can manually force the sync direction with the `--push` and `--pull` options on the server [ **TODO** not yet implemented]

You can disconnect (stop xCodea, quit Codea altogether, put the iPad to sleep, take the iPad on a long trip) whenever you want; the next time xCodea connects it'll sync any intervening changes and restart the project. There's no need to ever stop and restart the server unless you want to work on a different project (to stop the server, press ctrl-C in its terminal window).

### Live coding

Whenever a project file is changed on disk (i.e. on save) its contents are sent to the client. 

- If the file is not a valid lua chunk (meaning, there's a syntax error) you'll immediately see an error (with filename and line number).
- Otherwise the file will be evaluated inside xCodea's sandbox, updating your functions etc., and letting you see the effects of the changes almost immediately.
- Whenever a runtime error happens you'll see the stack trace (with filename and line number of the offending code).
	- If the error happens outside of the draw() event, your project will keep drawing in the background.
- Errors are displayed both on your iPad and in the server log.

Moreover the server watches for a special `eval.luac` file in your projectsRoot folder. As soon as this file is created, the server sends its contents to be evaluated inside xCodea's sandbox, then deletes the file. 

- You can use this as a live REPL to control and alter your project at runtime.
- You can send `xCodea.restart()` to safely reset the sandbox and restart the project execution
Finally, if you're using [LDT](http://www.eclipse.org/koneki/ldt/) (or some other Eclipse-based IDE) xCodea monitors the `.buildpath` file for your project to detect dependency changes (when a change occurs the project is automatically restarted).

### Tips

- Set up keyboard shortcuts to take advantage of `eval.luac`. For example I use [BetterTouchTool](http://www.boastr.net/) to intercept some key combinations while in LDT:
	- ⌘⏎: sends ⌘C, then executes `pbpaste > path/to/projectsRoot/eval.luac` (eval the current selection)
	- ⌘⌥⏎: executes `echo "xCodea.restart()" > path/to/projectsRoot/eval.luac` (restart the project)
- If you want to see the value of a variable or expression you can send it "naked" to `eval.luac`, xCodea will do the required `print()` wrapping (and sending to the server log) for you.
- When defining new classes or containers, use the idiom `myClass = myClass or class()` or `myObj = myObj or {}`. If you use `myClass = class()` myClass will be recreated every time the file is evaluated (which is every time the file is saved), destroying its state. Similarly, you should avoid initialising variables in the main chunk of a file - use setup functions (called in turn by the main `setup()`)
- [ **TODO** LDT, with link to Codea execution environment]
- [ **TODO** if the update() hook gets implemented, explain it here]

### Acknowledgements

xCodea was inspired by, and strives to improve upon, the excellent [LiveCodea](https://github.com/tofferPika/LiveCodea), which [begot AirCode](http://www.twolivesleft.com/Codea/Talk/discussion/comment/23225#Comment_23225) in Codea itself, whose limitations xCodea attempts to overcome. Some ideas were taken from the also excellent [AirCodea](https://github.com/CodeSturgeon/AirCodea)

### Todo

- complete this readme :)
- push, pull commands [DONE, more testing needed]
- update() hook
- if possible (almost certainly not), hijack tween() to use the update() hook
- warn about files deleted server-side
- extend the sandbox coverage to 100%; loadstring() is fully sandboxed, but dofile() and require() currently (probably) break it
- asset management (waiting for Codea 2 (and hoping it won't break everything))
- project discovery (and creation server-side) (~~possibly~~ available in a future Codea 2.x update)