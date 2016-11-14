# Sunday, November 13th Meeting

Build System Changes:
	- Build subdirectories
	- Out-of-tree builds (eg, build artifacts go to bin/ instead of being with normal files)
	- Seperation of tests into test/ directory, consisting of both in-kernel tests (tests/kernel/)
	and usermode tests (tests/user/)
	- Should we move to a CMake system instead of a make system? Could require some annoying/complicated
	changes in order to properly use linker scripts and so on.
	- Set up a linter to check against our code style guidelines, or give people a "make format" command.

Testing:
	- Everyone should have some units tests covering the public API in the kernel, and if relevant,
	some user tests for integration tests.
	- How will in-kernel tests work?
	- How will in-user tests work?

Code Style Guidelines:
	- We're not adopting any particular code style, but we'll put out the following requirements:
	- Use classes anywhere physically possible; structs should only be POD types; if you don't know what
	a POD type is, just use a class.
	- Avoid global metadata except when absolutely neccessary (eg, instead of several global pieces of metadata,
	put the metadata into a class and have 1 global pointer to that class).
	- Naming conventions are upper-camel case for classes (eg, HelloWorld), but methods and variables are lower
	snake case (eg, hello_world or the_thing).
	- Private variables should be prefixed with a single underscore (eg, _hello_world). Getters should be the normal
	name of the variable (hello_world()); setters should be prefixed with a set_ (eg, set_hello_world()).
	- Use const and constexpr whenever possible (eg, mark getters as const, and utility methods as constexpr when possible).
	- Indentation is 4 spaces per level.
	- Braces on the same line, 1 space between closing parenthesis and brace; eg, egyptian style:
	```C
	uint32_t hello_world() {
		// Things
	}
	```
	- Keep lines under 120 characters.
	- Commenting in the public API should be very important; explanatory comments of what's going are also important.

Initial Codebase:
	- Gheith's starting codebase is relatively ugly, but there is a large familiarity factor as people have largely used
	it instead of expanding on their own.
	- Code commenting is also important; all public interfaces should be reasonably commented.

Per Project API Requirements:
	- *Global Constructors/Destructors*: This would be really useful, so it would be to our benefit to properly
	set up calling global constructors for other people to use.
	- *Heap Implementations*: No changes, public API is fixed.
	- *Thread Scheduling*: All of this can be done without messing with the public API or making backwards-compatible changes
	so this can start immediately.
	- *Multiple Threads per Process*:
		- Minor changes to the thread implementation (particularly in Thread::exit).
		- Large changes to Process, including modification of public API for adding/removing threads to the process.
		- Addition of syscall create_thread to create new threads for the current process.
		- A way to signify the "main" thread which incoming signals will be put on.
	- *Nonblocking System Calls*: More research required; addition of new nonblocking syscalls which should conform
	to the Linux-ish thing; eg, epoll and select. Notifying the user process of completion can be done either through
	signalling, or through some other form of pre-emptive interrupt.
	- *Signals*: A major component of user-level processes which may be the backbone of efficient implementation
	of other components.
		- Add method for signalling a process (eg, process->signal(original_process (maybe?), code)).
		- Add a table for signal handlers, and default signal handlers.
		- Set up a way to actually call the signal handler on the running process, either by
		an inter-proccessor interrupt (better) or doing it on pre-emption (meh).
	- *Security & User Login & Permissions*:
		- Change of process to include a running user & group.
		- Change of filesystem to include per-file/director user & group permissions.
		- Change implementation of syscalls to check permissions.
		- User login will probably be one of the last thing.
	- *Filesystem*:
		- Mostly self-contained.
		- Buffer Cache should be transparent to end-user.
		- Addition of directory-traversal methods through an iterator.
		- Clarification of multi-threaded semantics.
		- Clarification of the multiple-write or write-multiple read cases, as end result is currently
		undefiend with this.
		- Note that you're also responsible for the mkfs.bobfs program which actually makes user tests.
	- *Other Filesystems*:
		- Need to make a more generic filesystem interface than just a BobFS Node (eg, create an abstract "Filesystem"
		and abstract "File" interface which we can query and resolve paths against).
	- *Shell*:
		- Not dependent on too many new features, can start working on user-mode implementation almost immediately.
		- Implementation of more advanced Ctrl+C features or other advanced keyboard functionality requires keyboard and mouse
		for which research needs to be done on (how to implement the drivers, and how to tell processes on inputs, and how to choose
		which process to give the input to).
		- Shell piping requires adding a pipe file handle type.
		- Look into support for file descriptor redirection (some kind of syscall?).
	- *GUI*:
		- Basic GUI details can be implemented in tandem with important shell research, in particular, how to send process and mouse events,
		how to handle the windowing system (eg, X server style, wayand compositor, etc).
		- Implementation of VGA buffer abstraction should be seperate from other code and relatively straightforward.
		- Standards for how to write to buffer, how often to update the VGA buffer, which process to let write to the VGA buffer, and so on.
	- *Porting Languages/Libc/other programs*:
		- Addition of syscalls as required by the newlib/libc implementation, which should be reasonably straightforward.
		- Should be doable immediately.
	- *Network Support*:
		- Requires a lot of research to get working properly, should be doable immediately though.
		- Potentially adding a file handle type for sockets.
		- Potentially a lot of hardware driver stuff for the qemu network card.
	- *Memory mapping*:
		- Changes to the address space to support the memory map data structures, and addition of methods for modifying said
		memory map datastructures.
		- Reimplementation of handling pagefaults to use the memory map.
		- Reimplementation of elf loading to use mmap instead of manual memory loading.
		- Implementation of mmap syscall.
	- *Other Virtual Memory*:
		- Demand Paging/Swapping: Should be mostly transparent to the user; addition of some global 
		metadata for actually determining stored pages; likely will go with Copy on Write.
			- Demand paging for only allocating pages when neccessary also under this.
		- Copy-on-write: Should only mess with some global address space structures, may need to work with
		Swapping and Demand paging to work correctly.
	- *I/O Drivers*:
		- Backbone of several other parts like the shell and graphical interface.
		- Implementation of mouse, keyboard, mass storage should be doable almost immediately; need to do research on how
		mouse and keyboard input should be handled/who it should be sent to.
	- *Real Hardware*:
		- This should require no algorithmic changes and can start immediately, but beware that you'll be dealing with
		almost non-stop race conditions and will need to make modifications to locking structures.
	- *Shutdown*:
		- Research/analysis for what needs to be saved/updated on shutdown would be useful.

Project Prioritization and Requirements:
	- Some projects can begin operations/implementation immediately, such as thread scheduling, libc implementation,
	permissions, BobFS improvements/filesystem improvements, some shell functionality, network support.
	- Some projects depend on a lot of other components, such as the GUI, busybox porting, and some networking features.
	- A few projects can significantly change the OS operation and need to be finished/prototyped almost immediately, such
	as signalling and mmap.

	- People on projects which can start immediately should start working immediately on getting barebones implementations done;
	try to not modify or change existing public APIs, but additions to any public APIs or additional syscalls should all be fine.
	- People on projects which rely on other projects should implement the abstractions and parts that they CAN; eg, for GUI,
	people can immediately start setting up drawing and messing with the VGA buffer, or do research on how to set up the windowing
	system.
	- People on core projects such as signalling and mmap should work with system architects to get a "minimum viable implementation"
	which other people can depend on for their projects, before going back and improving on the work.

Overall Changes:
	- Increase open file, semaphore, and process limits from 10 -> 128 (may need more efficient storage method for this).

What to tell people:
	- Show this document.
	- Have until Monday @ Noon to pick a project; can add themselves to additional optional projects to help on.
	- Have until Wednesday @ Noon to describe what pre-reqs they have and how they'd like to change the public API;
	sorry for the tight schedule, people need some time to present something on Friday? We can invoke the Ghieth to
	see if this can be moved back or if the Friday presentations can be on theoretical work/etc.

Contributing Guidelines:
	- "master" is the mainline branch, contains integrated code which actually works. Only working code which pasts tests
	should actually be commited to master.
	- Every group works on _feature branches_, which are branches off of master containing their specific code. When merging
	into master, people should always first rebase master into their local branch, fix any errors, and then merge their local
	branch back into master. Details on how to do this will be added into a CONTRIBUTING.md file.
	- Integration tests will be run on commits/merge requests to master.
