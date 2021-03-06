% title: Eclipse Plug-in Development
% title_class:                  #empty, largeblend[123] or fullblend
% subtitle: Extending the CDT Debugger
% author: Marc Khouzam
% author: Marc-André Laperle
% thankyou: Thank you
% thankyou_details:

---
title: Agenda
class: nobackground
build_lists: false
content_class:

- Day 1: Plug-in development
    - Module 1: Eclipse Platform and plug-in development
    - Module 2: Implementing a first plug-in
- Day 2: Getting to know DSF
    - Module 3: DSF concepts and exercises
    - Module 4: DSF events and exercises

---
title: Agenda (cont)
build_lists: false

- Day 3: Getting to know DSF-GDB
    - Module 5: DSF-GDB concepts and exercises
    - User meeeting
- Day 4: Modifying DSF-GDB's behaviour
    - Module 6: Changing the Debug View
    - Module 7: Changing GDB's initialization
    <br><br>
##<center>Let's get started!</center>
[//]: (This is a comment)

---
title: Approach to course

- A mix of theory and hands-on exercises
- Teams of 2
- Ask questions often and give feeback
- 9h to 16h schedule
- One hour lunch
- 15 minute break, morning and afternoon
- Please tell us if you are confused during the course

---
title: Start your Eclipse

- Select workspace: *~/workspace-training*

---

<br><br>
<br><br>
<br><br>
<br><br>
##<center>Plug-in development</center>

---
title: Module 1
subtitle: Eclipse Platform and Plug-in Development

- Intro to Eclipse Platform
<br><br>
- Plug-in development
<br><br>
- Exercises

---
title: The Eclipse Platform

Frameworks and Common services for:

- Rich Client Platform (RCP)
- Tool integration

<br/>
Examples of rich client applications:

- Eclipse for Java Developpers, Eclipse SDK, Stand-alone C/C++ Debugger, Trace Compass, etc.

Examples of tool integrations:

- CDT, EGit, Trace Compass plugins

---
title: The Eclipse Platform

<center><img src="images/sdk-arch.jpg"/></center>

---

title: Eclipse Runtime

Mainly composed of the Eclipse Equinox project.

Eclipse Equinox:

- Implements the OSGI specification (modules, services, etc)
- P2: Provisioning framework. Installs new plug-ins, resolves dependencies, etc.
- Miscellaneous Core functionallity like the extension registry (plug-ins), Jobs, Preferences, etc.
- Eclipse launcher (native)

<center><img src="images/EclipseRT.png"/></center>

---

title: Workspace

- The workspace is a view of the user's data files: the resources.
- Types of resources: projects, folders, and files
- Only one workspace per running platform
- Structure: Workspace root > Projects > IFolders or IFiles
- Common classes:
	- IResource
	- IWorkspace
	- IFile
 	- IFolder
	- IProject
- Resource names and their paths in the workspace to not have to match the location on the filesystem

---
title: Workspace

Other features than navigation/listing:

- Markers: to display problems like compilation errors
- Properties: per-file storage. Think flags like "read-only" but more customizable.
- Virtual folders: for greater flexibility when organizing projects
- Project natures: to differentiate between different kind of projects (Java, C/C++)
- Resource deltas: Listen to file modification, creation, deletion, etc.

---
title: EFS

Eclipse File System (EFS).

- An abstract file system API
- The Workspace resources sit on top of EFS
- The default implementations is for Local file systems
- Other implementations: SSH, FTP, dstore, ZIP, etc.

<center><img src="images/efs_project.png" width="50%" height="50%"/></center>

---
title: Eclipse Team

Framework to integrate SCMs.

- Repository configuration
- Resource management: Hooks for delete, move, add. Decorators in the UI.
- Synchronization: tracks whether or not resources are in sync. Local history.
- Logical Model Integration: Operate at the model level instead of per-file.
- Views: Synchronize, History, etc.

EGit, CVS, Subversion, Perforce use this.

<center><img src="images/egit.png"/></center>

---
title: Eclipse Help


- Table of content, Search, indexed
- Context sensitive (F1). Plug-ins can set context "ids"
- Runs on a local server, either in internal browser (SWT wiget) or external

Other "User Assistance" features

- Cheat cheets
- Welcome page
- Tutorials

<div style="position: absolute; left:400px; top: 400px;"><img src="images/welcome.png" width="70%" height="70%"/></div>
---
title: SWT

Standard Widget Toolkit (org.eclipse.swt.*)

<img src="images/vis-example.png"/>
<img src="images/lin-example.png"/>
<img src="images/mac-example.png"/>

A Java library of widgets aimed at providing efficient, portable access to the user-interface facilities of the operating systems on which it is implemented.

This is typically the lowest level of UI programming done in Eclipse development.

---
title: SWT
subtitle: Features

- Native look
- Quite fast!
- Button, Text, Browser, Group, Table, Tree, etc.
- Parent is specified in constructor (versus add child to parent container explicitly)
- Can be used as a stand-alone Java library (without anything else Eclipse)

---
title: SWT
subtitle: Implementation

- Single UI thread
- Implemented using JNI calling the OS native library (which means SWT has some native glue code)
	- Windows: Win32
	- Mac: Cocoa
	- Linux: GTK2 and GTK3
- Browser widget is integrated with different libraries: Webkit, Internet Explorer, XULRunner (Firefox)
- Most crashes (i.e. segmentation fault) in Eclipse due to native libraries called by SWT
	- GTK, WebKitGTK+, Ubuntu-specific libraries (Unity)
- A bit hard (but fun) to debug. Java + Native C
---
title: JFace

JFace is a UI toolkit with classes for handling many common UI programming tasks. It is designed to work with SWT without hiding it.

org.eclipse.jface.*

Common uses:

- Viewers: TreeViewer, TableViewer, etc.
- Dialogs: Message dialog, error dialog, etc.
- Wizards: Wizard dialog, pages

---
title: Workbench

org.eclipse.ui.*

This is where classes more related to the common IDE UI fonctionality reside.

<center><img src="images/workbench_decomposed.png" width="500" height="400"/></center>

---
title: Workbench

- A <b>Workbench</b> has one or more workbench windows.

- Each Workbench <b>Window</b> has workbench pages.

- Each Workbench <b>Page</b> has workbench parts, of which there are two kinds: views and editors.

- Each Workbench <b>Part</b> is either a <b>View</b> or an <b>Editor</b>.

<br/>
<center><img src="images/workbench_dia.png"/></center>

---
title: Workbench

Common uses:

- Views: IViewPart and the extension point org.eclipse.ui.views
- Commands and Handlers: org.eclipse.ui.commands, org.eclipse.ui.handlers extension points.
- Preference pages: org.eclipse.ui.preferencePages extension point
- Editors: org.eclipse.ui.editors extension point. IEditorPart and useful classes like TextEditor

---
title: The Eclipse SDK

To be able to work program with the Eclipse Platform, you need tools.

<img src="images/eclipse-logo-without-text.png" width="64" height="64"/>=<img src="images/jdt.png"  width="64" height="64"/>+<img src="images/plugins.png"  width="64" height="64"/><br/>

The Eclipse SDK = Platform (including source) + JDT + PDE

---
title: JDT: Eclipse Java development tools

Those are the tools suitable for any Java-based programming

<center><img src="images/editor_vectortest.png"/></center>

---
title: PDE: Plug-in Development Environment

Provides tools to create, develop, test, debug, build and deploy Eclipse plug-ins.

<img src="images/eclipse-pde.png"/>

---
title: Module 2
subtitle: Implementing a First Plug-in

---
title: Project for exercices

We will create a new view that will display a log of every function:line that the debugger stops at.
<center>
<img src=images/FrameSpy.png>
</center>
---
title: What IS an Eclipse plug-in?

It's an OSGI bundle, a java module.
But with an Eclipse flavor. Among other things:

- Specifies dependencies to other plug-ins
- Uses extensions to plug into existing extension points
- Can define new extension points for others to extend 
- Specifies what to package in the (Jar)
- Specifies the execution environment (Java 7, 8, etc).

Difference between extension and extension point?

Extension = plug<br/>
Extension point = socket

A lot of things are done through extension points. For that, we need a plug-in.

---
title: Exercise: Create a plug-in

- Go to Plug-in Development perspective
- File > New > Plug-in project
- Name your plug-in (org.eclipse.cdt.example.framespy)
- Press Next then Finish
- <b>Go!</b>

What are all the tabs for?

<center><img src="images/manifest_tabs.png"/></center>

---
title: Step to prepare

- Since we want to use the Git repo, delete the project and import existing one.
    - Right-click on your project and press Delete then press OK
- Go to *Git perspective*
    - In *Git Repositories* right-click and choose *Remove Repository from View*

---
title: Step to prepare (2)

- Go to *Plug-in Development perspective*:
    - *File->Import...->General->Existing Projects into Workspace*
    - Press *Next*
    - Press *Browse...*
    - Choose *~/workspace-training/EclipseTraining/org.eclipse.cdt.example.framespy*
    - Make sure a single project is showing and is selected
    - Press *Finish*

- Right-click on project and choose *Team->Fetch from Upstream*

---
title: Exercise: Create a view

- Reset to **PLUG1**
- Add a new view by adding an extension (plugin.xml)
- Create the view class (tip: click the hyperlink to bring up the New Class wizard)
- Make sure the view has an id, name
- To test your progress 

<center> <img src="images/LaunchCourse.png" width="400"/></center>

- <b>Go!</b>

---
title: SWT Basic widget creation

<pre class="prettyprint" data-lang="java">
Text textBox = new Text(parentComposite, SWT.SOMESTYLE | SWT.OTHERSTYLE);
textBox.set*Somesetter*
textBox.add*Somelistener*
</pre>

Composite are containers of widgets that can be layed out.

Composites can be in composites!

---
title: Exercise: Add button to view

Let's add a Button. Let's make it a checkbox button.

- Reset to **PLUG2**
- In view class, in createPartControl() add new Button with a "checkbox style"
- Add selection listener to detect when it's pressed
- Output something to the console
- <b>Go!</b>

But the button does not belong in view itself, it would be nicer in the toolbar. <img src="images/quick_fix.png" width="50" height="40"/>
---
title: Eclipse commands and handlers

The Commands framework is a way to add user actions to the user interface.
A command has:

- An id that declares a semantic behavior
- One or more handlers

For example, the Copy command has the id *org.eclipse.ui.edit.copy*. But it has many handlers that behave differently depending on the context (selection, view)

---
title: Exercise: Create the command and handler

- Reset to **PLUG3**
- Add a command extension to toggle log on/off
- Create a defaultHandler class (tip: use hyperlink to create class). Make it extend AbstractHandler (the one in 'core')
- Implement the execute method to make it print something to the console
- <b>Go!</b>
---
title: The menus extension point

The org.eclipse.ui.menus extension point can add commands to:

- Main menu (menu)
- Main toolbars (toolbar)
- View context menus (popup)
- View toolbars (toolbar)
- View menus (menu)

It has an odd **locationURI** field: &nbsp;<span style="background-color:#eeeeee;">[Scheme]:[ID]?[placement]</span>

For example:

<pre class="prettyprint">
menu:org.eclipse.ui.main.menu?after=additions
</pre>

---
title: Exercise: Add toolbar button

Let's add the command to the view toolbar.

- Reset to **PLUG4**
- Create the menus extension (Extension tab, org.eclipse.ui.menus extension)
- Add a menuContribution, set the locationURI so that it gets displayed in the view toolbar, after additions
	- locationUri: toolbar:my.view.id?after=additions
- Right-click on menuContribution, Add a command. Set the id, icon and style
	- id: the id of your command
	- icon: enablespy.gif
	- style: push
- <b>Go!</b>

It would be nice in the context menu as well. <img src="images/quick_fix.png" width="50" height="40"/>

---
title: Exercise: Context menu

Let's add the same command in the context menu.

- Reset to **PLUG5**
- Add a new menuContribution (right-click on org.eclipse.ui.menus). Because it needs different locationURI. 
	- locationUri= popup:my.view.id
- Add the command (right-click on menuContribution). Set the id, icon and style (push)
- Create a context menu in the view, to be populated:

<pre class="prettyprint" data-lang="java">
// put in createPartControl
fMenuManager = new MenuManager();
Menu menu = fMenuManager.createContextMenu(composite);
composite.setMenu(menu);
getViewSite().registerContextMenu(fMenuManager, null);
...
// put in dispose (override the method)
fMenuManager.dispose();
</pre>

- <b>Go!</b>
---
title: Exercise: Show the toggle state

The user needs to see whether logging is enabled or not.

Let's add a label that displays that.

- Reset to **PLUG6**
- Add a Label (a SWT widget) at the view creation.
	- Label label = new Label(c, SWT.NONE)
	- label.setText(Boolean.FALSE.toString());
- When the command executes:
	- Set the toggle state (you can add a method to the view)
	- In the handler, get the view with FrameSpyView part = (FrameSpyView) HandlerUtil.getActivePartChecked(event);
	- Update the label text
		- label.setText

- <b>Go!</b>


If you enable logging and close/reopen the view, what happens?
---
title: The Preference Store

We should make sure that the toggle state is remembered when the view is closed (or Eclipse is restarted). There are multiple ways to do this. A useful one is the Preference Store. 

- Works like a key/value map
- Can be applied to different scopes:
	- DefaultScope: When the user presses "restore defaults" it restores to this.
	- InstanceScope: Saved at the workspace level. Overrides Default.
	- ProjectScope: Saved at the project level. Overrides Instance.
	- Custom!
- Organized in nodes (think namespaces). Typically the plug-in id.
---
title: Exercise: Persist the toggle state

- Reset to **PLUG7**
- When the toggle state is set:
	- Get the InstanScope
	- Get the node
	- Set the key/value

Saving the preferences in our FrameSpyView.togleState method

<pre class="prettyprint" data-lang="java">
IEclipsePreferences preferences = InstanceScope.INSTANCE.getNode(Activator.PLUGIN_ID);
preferences.put(TOGGLE_STATE_PREF_KEY, Boolean.toString(fToggledState));
</pre>

---
title: Exercise: Persist the toggle state

- When the view is created:
	- Get the InstanScope
	- Get the node
	- Get value using key, set the label text

<pre class="prettyprint" data-lang="java">
IEclipsePreferences preferences = InstanceScope.INSTANCE.getNode(Activator.PLUGIN_ID);
preferences.get(TOGGLE_STATE_PREF_KEY, Boolean.toString(false)); // set to false in case it was never set
</pre>

- <b>Go!</b>


---
title: Eclipse Jobs

Jobs are similar to Java Thread but have Eclipse flavor.

Some differences:

- Scheduling rules: determine which jobs can run concurrently
- Deadlock detection: and recovery (ILock)
- Shown in the Progress View to the user (or not) with progress (IProgressMonitor)
- Can be made cancellable by the user
- Returns a status (IStatus)

In Eclipse code, Jobs and Threads are both commonly used, depending on the situation.

For the first implementation of our logging feature, we will poll every one second using a job.

---
title: Eclipse Jobs

<pre class="prettyprint" data-lang="java">
job = new Job("Frame Spy Polling Job") {
	@Override
	protected IStatus run(IProgressMonitor monitor) {
		return Status.OK_STATUS;
	}

};
job.schedule();
</pre>

---
title: Exercise: Creating a polling job

- Reset to **PLUG8**
- When toggle state is on, create and schedule a job
	- Give the job a nice name to be shown in the Progress View
- In the job run() method, sleep for 1 sec. Use Thread.sleep
- After the 1 sec, reschedule the job (only if the toggle state is still on!). Use schedule()
- Print something to the console every tick "polling..."
- If the toggle state becomes 'off' cancel the job. Use job.cancel().
- <b>Go!</b>

---
title: Exercise: Print to view

Every tick, we would like to show the elapsed time in the view. It can just be a 

- Reset to **PLUG9**
- Create a counter (a simple int) that is incremented each tick.
- Set the text label with the value of the counter
- <b>Go!</b>

What happens?
---
title: SWT Display and the UI thread

Changes to the UI (widgets) should always be done on the UI thread.

The Display implements the event loop. There is only one instance in a running Eclipse.
 
<pre class="prettyprint" data-lang="java">
Display.getDefault().asyncExec // Execute code at the next reasonable opportunity. Caller continues in parallel.
Display.getDefault().syncExec // Blocks calling thread until executed on UI thread
</pre>

With this knowledge we can fix the Invalid Thread Access. We can use asyncExec in this case. (**PLUG10**)
---
title: Exercise: Handle cancel

An IProgressMonitor is passed to the job.

We can use it to know if the user canceled the logging.

- Reset to **PLUG10**
- Use monitor.isCanceled() to know when user canceled
- Set the toggle state to off when canceled
- <b>Go!</b>

What happens? Do you have a problem with the job starting again?
If you need results from UI thread right away -> syncExec

---
title: Exercise Review
subtitle: What we accomplished

- Create a plug-in
    - *File > New > Plug-in project*

<center><img src="images/manifest_tabs.png"/></center>

---
title: Exercise Review (2)
subtitle: What we accomplished
build_lists: false

- Creating a view
    - *extension point="org.eclipse.ui.views"*
    - Extending *ViewPart* and filling *createPartControl()*

- Using SWT widgets
    - Using *Button*, *Label*
    - Using the UI Thread for such things
    - *Display.getDefault().syncExec()/asyncExec()*

---
title: Exercise Review (3)
subtitle: What we accomplished
build_lists: false

- Using commands, handlers and menus
    - *extension point="org.eclipse.ui.commands"*
    - Extending *AbstractHandler#execute()* to do the work of the command
    - *extension point="org.eclipse.ui.menus"*
        - *toolbar:org.eclipse.cdt.example.framespy.view?after=additions*
        - *popup:org.eclipse.cdt.example.framespy.view*
    - Initializing a context-menu using *MenuManager*
    
---
title: Exercise Review (4)
subtitle: What we accomplished
build_lists: false

- Using the preference store
    - *InstanceScope.INSTANCE.getNode(PLUGIN_ID)*
- Using Jobs and Progress Monitors
    - *Job#schedule()*
    - *monitor.isCanceled()*

---

<br><br>
<br><br>
<br><br>
<br><br>
##<center>Getting to know DSF</center>

---
title: Module 3
subtitle: DSF Concepts and Exercises

<br><br>
- Eclipse Debug Platform
<br><br>
- What is DSF?
<br><br>
- DSF concepts applied
<br><br>
- DSF Exercise 1

---
title: Building on our new view

- Debug Frame Spy
    - New view logging each `method:line` at which the debugger stopped. 

---
title: Debug Frame Spy Details

- Show list of `method:line` of each location the program was interrupted
- Show time of interrupt for each entry
- Show the number of arguments of the function for each entry

<center><img src=images/FrameSpy.png></center>

---
title: Eclipse Debug Platform

- Eclipse provides a foundation for Debugging
    - Debug perspective
    - Debug views
    - Debug Actions, Toolbar, Menus
    - Debug Launching

---
title: What is DSF?

- Overview
- View Model and Data Model
- Services
- Data Model contexts
- DSF Executor thread
- DSF Session
- Asynchronous (callback) programming

---
title: DSF Overview

- API for integrating debuggers in Eclipse
- Also designed for efficiency (slow or remote targets)
- Figure shows typical debugger integration using DSF

<center><img src=images/DSFOverview.png></center>

---
title: View Model

- View Model provides layer of abstraction for views
    - *User-presentable* structure of the data
<br><br>
- View Model allows to easily modify presentation e.g.,
    - Hide running threads
    - Limit number of stack frames
    - Only show processes if there is more than one

---
title: Using the CDT Debugger

- Start your test Eclipse
<br><br>
- Use the debug icon to debug a C program
<br><br>
- Step, look at variables, set breakpoints, resume

---
title: Data Model

- Data Model deals directly with the backend debugger
    - *Natural* or *backend* structure of the data
    - Independent of presentation to user
    - Provides building blocks for the view model
    - Uses common debugger concepts
        - Execution elements (e.g., processes, threads)
        - Formatted values (e.g., variables, registers)
        - etc

---
title: DSF Services

- DSF provides a service API to access the Data Model
<br><br>
- Built on top of OSGi (as Eclipse is)
<br><br>
- Services are entities managing logical subsets of the data model
<br><br>
- Services are used to request information or to perform actions
---
title: DSF Services (2)

- For example, the IRunControl service:
    - Provides list of execution elements (e.g., threads, processes)
    - Provides details about such elements (e.g., name, state)
    - Supports step, resume, interrupt, etc
<br><br>
- Other services: IMemory, IRegisters, IExpressions, IDisassembly...
<br><br>
- All services extend <code>IDsfService</code> (press *F4* on <code>IDsfService</code>)

---
title: Data Model Contexts

- IDMContext class is a 'pointer' to any type of backend data
    - IExecutionDMContext - thread, process, group
    - IFrameDMContext - stack frames
    - IBreakpointDMContext - breakpoint, tracepoint, dprintf
    - All contexts extend <code>IDMContext</code> (use *F4*)
<br><br>
- Contexts are hierachical
    - *process* -> *thread* -> *frame* -> *expression*
    - <code>DMContexts.getAncestorOfType()</code>
<br><br>
- Contexts are used to retrieve data from services

---
title: DSF Executor thread

- Accessing data from different threads requires synchronization
- DSF uses a single-threaded executor to avoid synchronization
<center>
<img src=images/synchronization_1.png>
<img src=images/synchronization_2.png>
</center>

---
title: DSF Session

- Instances of DSF services are grouped into a DSF session
<br><br>
- There can be multiple sessions running at the same time
<br><br>
- The session provides the DSF Executor (<code>DsfSession#getExecutor()</code>)
<br><br>
- A session handles sending events to registered listeners

---
title:  Asynchronous (callback) programming

- Most DSF APIs return void but indicate completion in a callback
<br><br>
- <code>RequestMonitor</code> is the main callback class
    - Remember to call <code>done()</code> when real work is finished
    - This calls: <code>handleCompleted()</code>, <code>handleSuccess()</code>, <code>handleError()</code>
<br><br>
- <code>DataRequestMonitor</code> to *"return"* a value
    - <code>getData()</code> to get that value

---
title: RequestMonitor example

- To call an asynchronous method, such as:

<pre class="prettyprint" data-lang="java">
void asyncCall(IDMContext dmc, RequestMonitor rm);
</pre>

- there are two main coding styles:

Declarative:

<pre class="prettyprint" data-lang="java">
   RequestMonitor myRm =
           new RequestMonitor(getExecutor(), parentRm);
   asyncCall(dmc, myRm);
</pre>

In-line:

<pre class="prettyprint" data-lang="java">
   asyncCall(dmc, new RequestMonitor(getExecutor(), parentRm));
</pre>
---
title: Declarative Style

- First declare the RequestMonitor and what it should do
<br><br>
- Then call the asynchronous method, passing the RM

<pre class="prettyprint" data-lang="java">
   RequestMonitor myRm =
           new RequestMonitor(getExecutor(), parentRm) {
               @Override
               void handleSuccess() {
                   System.out.println("Async call succeeded");
                   parentRm.done();
               }
           };

   asyncCall(dmc, myRm);
</pre>

---
title: In-line Style

- Directly call the asynchronous method
<br><br>
- Declare and define the RM in-line 

<pre class="prettyprint" data-lang="java">
   asyncCall(dmc, 
             new RequestMonitor(getExecutor(), parentRm) {
                 @Override
                 void handleSuccess() {
                     System.out.println("Async call succeeded");
                     parentRm.done();
                 }
             });
</pre>

- *In-line* has the benefit of showing the execution flow

---
title: DataRequestMonitor

- Extention of <code>RequestMonitor</code> which *"returns"* data

<pre class="prettyprint" data-lang="java">
   DataRequestMonitor<String> parentRm =
              new DataRequestMonitor<String>(getExecutor, null);
   asyncCallWithData(
      dmc, 
      new DataRequestMonitor<String>(getExecutor(), parentRm) {
      @Override
      void handleSuccess() {
          String resultString = "Success with result " + getData();
          parentRm.done(resultString);
      }
   });
</pre>

---
title: Other RequestMonitors

- <code>CountingRequestMonitor</code> and <code>MultiRequestMonitor</code>
    - For multiple asynchronous request in parallel
<br><br>
- <code>ImmediateRequestMonitor</code> and similar
    - <code>handleSuccess()</code> and others are called on the thread where the ImmediateRM was created.

---
title: DSF concepts review
build_lists: false

- APIs to integrate a debugger 'more easily' e.g., GDB
- View Model for presentation layer
- Data Model to communicate with backend (GDB)
- Services API to access Data
- No synchronization: DSF Executor **must** be used to access Data
- Services for one backend are grouped in a Session
- Heavy use of asynchronous programming for responsiveness

---
title: DSF practical review
build_lists: false

- Services extend <code>IDsfService</code>
<br><br>
- Contexts extend <code>IDMContext</code>
<br><br>
- Context hierarchy searched with <code>DMContexts</code>
<br><br>
- Executor can be found with <code>DsfSession#getExecutor()</code>
<br><br>
- <code>RequestMonitor</code> and <code>DataRequestMonitor</code> for callbacks

---
title: DSF Exercise 
build_lists: false

- FrameSpy to periodically print "method:line" for current frame 
    - Reset branch to commit starting with
        - **DSF1_START** or **DSF1_ADVANCED**
<br><br>
    - To test, make sure you launch a C/C++ Debug session first
<br><br>
    - Use the **Tasks** view to see what needs to be done
<br><br>
    - **Go!**

---
title: Exercise review
build_lists: false

- Finding the DSF session using debug context
    - Debug View and Debug Context
    - Adapter pattern
<br><br>
- Calling an existing DSF service
    - Using a DsfServicesTracker for the DSF session
<br><br>
- Call the asynchronous IStack.getTopFrame()
    - Using a new DataRequestMonitor
    - Calling getData() in handledSuccess()
---
title: Exercise review (2)

- IDMContext vs IDMData
    - call IStack.getFrameData()
    - Using a new DataRequestMonitor
    - Calling getData() in handledSuccess()
    - Then finally display "method:line"

---
title: Exercise follow-up part 1
build_lists: false

- What if you select the process element?
    - The top frame of which thread should we use?
<br><br>
- For now, just handle the error (as seen on console)
    - Reset to **DSF1_ANSWERS** if you need
    - Override <code>handleError()</code>
    - **Go!**

---
title: Follow-up part 1 review

---
title: Exercise follow-up part 2

- Assertions are a great way to notice unexpected situations
<br><br>
- Enable assertions in development eclipse for test eclipse
    - In launch configuration, *Arguments* tab, *VM arguments*
    - Add *-ea*
    - Make sure you have a breakpoint for *AssertionError*
    - Re-launch and try
    - **Go!**

---
title: Exercise follow-up part 2

- Did you use the DSF Executor?
    - Which code runs on the Executor, which not?
<br><br>
- Wrap first call to DSF service in Executor
    - Call <code>submit()</code> of the Executor
    - Pass a <code>DsfRunnable()</code> whose <code>run()</code> does the work
    - **Go!**

---
title: Follow-up part 2 review

---
title: Module 4
subtitle: DSF Events and Exercises

- What are DSF events
<br><br>
- Sending and receving DSF events
<br><br>
- DSF Exercises 2 and 3

---
title: DSF Events

- DSF uses events to notify listeners of different things e.g.,
    - Thread/Process started/exited
    - Thread/Process suspend/resumed
    - Breakpoint added/updated/removed
    - etc

---
title: DSF Events (2)
build_lists: false

- Events are how the Data Model tells the View Model of changes
    - e.g., Thread stops => Update Debug View
    - View Model is an advanced topic not covered in this course
<br><br>
- Events also notify services of other services' changes
    - e.g., Clearing caches when execution resumes
---
title: DSF Events details

- Most events implement <code>IDMEvent</code> which provides an <code>IDMContext</code>
    - e.g., When thread suspends, event specifies which thread
<br><br>
- Event types usually found in the different service interfaces e.g.,
    - <code>IRunControl</code>:
        - <code>ISuspendedDMEvent</code>, <code>IContainerSuspendedDMEvent</code>
        - <code>IResumedDMEvent</code>, <code>IContainerResumedDMEvent</code>
<br><br>
- Not all services trigger events
    - <code>IStack</code> has not events
---
title: Sending DSF Events

<br><br><br>
- To send an event a service calls <code>DsfSession#dispatchEvent()</code>
---
title: Receiving DSF Events

- To receive a DSF events a client must:
    - Declare a **public** method of any name
    - Method takes the event of interest as a parameter
    - Annotate method with <code>@DsfServiceEventHandler</code>
    - Register with the DSF Session using <code>DsfSession#addServiceEventListener()</code>
    - Registration must be done on the Executor
<br><br>
- The method is called on the DSF Executor

---
title: Receiving event example

- The following method from *SomeClass* will be called for every suspended event

<pre class="prettyprint" data-lang="java">
    @DsfServiceEventHandler
    public void anyName(ISuspendedDMEvent e) {
        System.out.println("Received " + e.toString());
    }
</pre>

- as long as we register the class with the session

<pre class="prettyprint" data-lang="java">
    getSession().addServiceEventListener(SomeClass.this, null);
</pre>

- Remember that registration must be done on Executor
---
title: Help with the Executor
build_lists: false

- DSF provides Java Annotations to guide with Executor use 
    - <code>@ThreadSafe</code>
        - Safe for any thread (synchronization used)
    - <code>@ConfinedToDsfExecutor(executor)</code>
        - Must use specified executor
    - <code>@ThreadSafeAndProhibitedFromDsfExecutor(executor)</code>
        - Safe for any thread **except** the specified executor
<br><br>
- They are hierarchical, so apply to children (e.g., methods of class)
<br><br>
- Unfortunetly, there is no compiler support so they are effectively just comments (that are sometimes missing)

---
title: DSF Event Exercise

- Show "method:line" each time a thread stops instead of polling
    - Reset branch to commit starting with
        - **DSF2_START** or **DSF2_ADVANCED**
        - Polling job has been removed for you
        - "method:line" only shown when FrameSpy first enabled
<br><br>
    - To test:
        - make sure your debug session is in Non-Stop mode
        - step program and check new "method:line"  each step
<br><br>
    - **Go!**

---
title: Event Exercise Review

- Registering for DSF events
    - addServiceEventListener() **using** the Executor
    - Must pass <code>FrameSpyView.this</code> (or another listener class)
<br><br>
- Unregister for DSF events when FrameSpy disabled
    - removeServiceEventListener() **using** the Executor
    - Must pass <code>FrameSpyView.this</code> (or listener used)
    
---
title: Event Exercise Review (2)

<br><br>
- Receiving the event

<pre class="prettyprint" data-lang="java">
@DsfServiceEventHandler
public void anyName(ISuspendedDMEvent event) {
    // Fetch frame info and print it
}
</pre>

---
title: Event Exercise for All-Stop

- <code>ISuspendedDMEvent</code> is used for Non-stop only
<br><br>
- <code>IContainerSuspendedDMEvent</code> for All-stop
    - Represents the process stopping
    - The top frame of which thread should we use?
<br><br>
- This event specifies which thread caused the stop
    - Use that **triggerring** thread (context)
    - (Look at declaration of <code>IContainerSuspendedDMEvent</code>)
    - Reset to **DSF2_ANSWERS**
    - **Go!**

---
title: All-Stop Exercise Review

---
title: Handling a new session

- FrameSpy has an important limitation now
    - enable FrameSpy
    - stop the session and start a new one
    - step the new session
    - **FrameSpy no longer prints**

---
title: Handling a new session (2)

<br><br>
<br><br>
<br><br>
##<center>**Why?**</center>

---
title: Handling a new session (3)
build_lists: false

- When new session starts, we are not registered for its events
<br><br>
- How to know **when** new session starts so we can register?

---
title: DsfSession to the rescue

- DsfSession notifies registered listeners of start/end of all sessions
    - <code>addSessionStartedListener()</code>, <code>removeSessionStartedListener()</code>
    - <code>addSessionEndedListener()</code>, <code>removeSessionEndedListener()</code>
<br><br>
- DsfSession provides access to all running sessions:
    - <code>getActiveSessions()</code>, <code>getSession(id)</code>

---
title: Multiple Session Exercise

- Register for event for each new DSF session
    - Reset to **DSF3_START** or **DSF3_ADVANCED**
<br><br>
    - Listen for new session and register with them
<br><br>
    - Unregister when FrameSpy gets disabled
<br><br>
    - **Go!**

---
title: Sessions Exercise Review

---

<br><br>
<br><br>
<br><br>
<br><br>
##<center>Getting to know DSF-GDB</center>

---
title: Module 5

- What is DSF-GDB
<br><br>
- A little history
<br><br>
- DSF-GDB's service structure
<br><br>
- DSF Exercises 4, 5 and 6

---
title: What is DSF-GDB

- Integration of GDB using DSF
    - Cannot use run DSF by itself
<br><br>
- Extra features on top of base DSF
    - Tracepoints
    - Visualizer
    - OS Resources

---
title: History of DSF-GDB

- How it started
<br><br>
- Ericsson's involvement
<br><br>
- GDB's evolution
<br><br>
- Default CDT Debugger integration
<br><br>
- Where we stand today

---
title: DSF-GDB's services

- DSF provides API for services
    - <code>IStack</code>, <code>IBreakpoints</code>, <code>IExpressions</code>, etc
<br><br>
- DSF-GDB provides an implementation
<br><br>
- Hierarchy of DSF-GDB services
    - Press *F4* on <code>IDsfService</code>
    - <code>MI[service]</code> vs <code>GDB[service]</code> (historical)
    - <code>GDB[service][version]</code>
    - <code>GDB[service]_HEAD</code>

---
title: New Service Exercise
build_lists: false

- Write a new service providing the current time
    - Reset to **DSF4_START** or **DSF4_ADVANCED**
<br><br>
    - **FrameSpyService.java** already created for you
<br><br>
    - Make it into a DSF service that can be found by name
<br><br>
    - Provide methods:
        - Synchronous <code>getLocalTimeOfDayString</code> method
        - Asynchronous <code>getTargetTimeOfDayString</code> method
<br><br>
    - **Go!**

---
title: New Service Review
build_lists: false

- <code>AbstractDsfService</code> can be used as a base class for services
<br><br>
    - Need to implement <code>getBundleContext()</code>
<br><br>
    - Need to advertise a service using <code>register()</code>
        - Any name can be used but class or interface name is good
<br><br>
    - <code>initialize()</code> and <code>shutdown()</code> should be enhanced
<br><br>
    - Some method providing the service functionality is needed

---
title: Asynchronous vs Synchronous API

- Slowest part of the CDT debugger is communication with GDB
    - DSF provides infrastructure for async communication
    - New async API can use that infrastructure
    - New sync API **cannot**
<br><br>
- Async API can be used synchronously but not other way around

---
title: Using new service

- Prepend every printout in FrameSpyView with the time of day
    - Reset to **DSF4_UPDATE_START**
<br><br>
    - FrameSpyView to show:  [time] method:line
<br><br>
    - If you test it, it will **not** work (yet)
<br><br>
    - **Go!**

---
title: Instantiating new service
build_lists: false

- We can't find the service because we didn't instantiate it
<br><br>
- We need one instance for **each** DSF session
<br><br>
- A DSF-GDB session instantiates its services
    - We haven't hooked into a DSF-GDB session (yet)
    - We need to manage our new service ourselves
    - Remember <code>addSessionStartedListener()</code> and friends?

---
title: Instantiation Exercise

- Implement a managing class to create/dispose of the new service
    - Reset to **DSF5_START** or **DSF5_ADVANCED**
<br><br>
    - Done for you:
        - Singleton **FrameSpyServiceManager.java**
        - <code>initialize()</code> and <code>dispose()</code> called by <code>Activator</code>
<br><br>
    - Instantiate and <code>initialize()</code> a FrameSpyService for each new DSF session
<br><br>
    - When done FrameSpyView should show:  [time] method:line
<br><br>
    - **Go!**

---
title: Service Shutdown

- We instantiate a service for each new DSF session
<br><br>
- What about shutting down those instances?
    - Each time a DSF session ends
- DSF-GDB automatically shutsdown **all** DSF services
    - Anything registered and implementing <code>IDsfService</code>
    - We don't need to take care of it ourselves
    - Refer to DSF-GDB's **ShutdownSequence.java**

---
title: Frame Argument count
build_lists: false

- Provide the number of arguments when printing "method:line"
    - Reset to **DSF6_START** or **DSF6_ADVANCED**
<br><br>
    - Provide API (method) in your service for arguments count
    - Async or Sync? 
        - Is the info needed in GDB?
        - Are you going to call any async APIs?
<br><br>
    - Use <code>IStack</code> service to get list of frame arguments
<br><br>
    - Update FrameSpyView to show:  [time] method:line (# args)
<br><br>
    - **Go!**

---
title: Exercise Review
---

<br><br>
<br><br>
<br><br>
##<center>Modifying DSF-GDB's behaviour</center>


---
title: Module 6
subtitle: Changing the Debug View

- Extending an existing service
<br><br>
- Launch Delegates and Launch Configuration Types
<br><br>
- Service Factories
<br><br>
- DSF Exercises 7 and 8

---
title: Building on DSF-GDB

- For a new debugging feature
<br><br>
    - Use existing DSF-GDB services
<br><br>
    - Re-work obtained information
<br><br>
    - Provide new information to view

---
title: Modifying DSF-GDB

- To change an *existing* debugging feature
<br><br>
    - Override View Model code
<br><br>
    - Override Data Model code
<br><br>
- We will focus on overriding a service (i.e., the data model)

---
title: Extending a service
build_lists: false

- For stack frames, replace method name "main" with "entry"
    - Reset to **DSF7.1_START** or **DSF7.1_ADVANCED**
<br><br>
    - **FrameSpyStackService.java** extends existing Stack service
<br><br>
    - Override <code>getFrameData()</code> 
<br><br>
    - "Return" an <code>IFrameDMData</code> whose <code>getFunction()</code> returns "entry" instead of "main"
<br><br>
    - **Go!**
---
title: New Service Instantiation
build_lists: true

- Like before we now have a new class we must instantiate
<br><br>
- Can we use <code>FrameSpyServiceManager?</code>
    - <code>FrameSpyStackService</code> extends existing <code>GDBStack_HEAD</code> service
    - <code>GDBStack_HEAD</code> is already being instantiated by DSF-GDB
    - Instantiating ours would create **two** IStack services
<br><br>
- We need to instantiate our service *instead* of the original one

---
title: Two approaches to extend DSF-GDB

- Creating a new view and a new service
    - Does not affect the rest of the debugging views
    - We can do this to any DSF-GDB session
    - This was our first set of exercises
<br><br>
- Replacing a service and changing an existing view
    - Does affect normal debugger behaviour
    - Should be chosen by user explicitly
    - Aimed at specific scenarios
    - This is what we need now

So, how do we *replace* a service?

---
title: First solution

- New *Launch Configuration Type*
    - Current ones
        i. *C/C++ Application*
        i. *C/C++ Attach to Application*
        i. *C/C++ Postmortem Debugger*
        i. *C/C++ Remote Application*
<br><br>
    - Add a new one such as "IMA2l-Chip Debugger"
<br><br>
    - Launch Config Types need a Launch Delegate
<br><br>
    - When chosen by user, we know to replace IStack service
         
---
title: Second solution

- New *Launch Delegate* to existing *Launch Configuration Type*
    - Current ones for *C/C++ Application*
        i. *GDB (DSF) Debug Process Launcher*
        i. *Legacy Create Process Launcher* (will be removed)
<br><br>
    - Add a new one such as "IMA2l-Chip Local Launcher"
<br><br>
    - When chosen by user, we know to replace IStack service

---
title: Differences

- Both solutions are almost the same
    - Both need a new launch delegate
    - The first also provides a new launch config type
    - The second re-uses existing launch config types
<br><br>
- Base choice on the UI presented to the user
<br><br>
- Code differences are minor

---
title: Launch Delegate exercise
build_lists: false

- Create a new Launch Delegate for *C/C++ Application*
    - Reset to **DSF7.2_START**
<br><br>
    - **FrameSpyLaunchDelegate.java** extends <code>GdbLaunchDelegate</code>
<br><br>
    - Update *Extensions* tab of *plugin.xml*
        - Fill-in <code>org.eclipse.debug.core.launchDelegates</code>
        - "Main" Launch tab has been provided for you
<br><br>
    - **Go!**

---
title: Launch Delegate Review

- When creating a "C/C++ Application" launch, we can now select our delegate
<br><br>
- But the new delegate does exactly what DSF-GDB does
<br><br>
- It still does not instantiate <code>FrameSpyStackService</code>

---
title: New Services Factory

- Now that we have a new delegate, we can create a new *Services Factory*
<br><br>
- A DSF Service Factory is used to create the different services
    - Let's have a look at <code>GdbDebugServicesFactory</code>
<br><br>
- When user chooses new delegate, they will get new IStack service

---
title: Services Factory Exercise
build_lists: false

- Create a new Services Factory for our Launch Delegate
    - Reset to **DSF7.3_START**
<br><br>
    - **FrameSpyServicesFactory.java** extends <code>GdbDebugServicesFactory</code>
<br><br>
    - This new service factory instantiates <code>FrameSpyStackService</code>
<br><br>
    - Our delegate uses <code>FrameSpyServicesFactory</code>
        - by overridding <code>newServiceFactory()</code>
<br><br>
    - **Go!**
<br><br>
    - You should be able to see "entry" instead of "main"

---
title: Current status

- We have a new delegate <code>FrameSpyLaunchDelegate</code>
<br><br>
- We have a new service factory <code>FrameSpyServicesFactory</code>
<br><br>
- We have a new service <code>FrameSpyStackService</code>
<br><br>
- New delegate uses new service which replaces "main" with "entry"

---
title: Service_HEAD pattern

- Why did our new service extend <code>GDBStack_HEAD</code>?
<br><br>
- Recent improvement allows extenders to stay on newest GDB version
<br><br>
- Let's look at an example
    - F4 on <code>GDBControl_HEAD</code>
- <code>GDBService_HEAD</code> always points at newest service version
<br><br>
- Favors stability of latest GDB version at the detriment of older ones

---
title: Launch Config Type exercise

- Create a new Launch Configuration Type for our delegate
    - Reset to **DSF8_START**
<br><br>
    - Use extension points in *plugin.xml*
    - <code>o.e.debug.core.launchConfigurationTypes</code>
    - <code>o.e.debug.ui.launchConfigurationTypeImages</code>
    - <code>o.e.debug.ui.launchConfigurationTabGroups</code>
<br><br>
    - Assign our launch delegate to new launch config type
<br><br>
    - Assign our launch tab to new launch tab group
<br><br>
    - **Go!**

---
title: What we've seen

- We've created a new view that used existing services
<br><br>
- We've created a new service that the new view can use
<br><br>
- We've created a replacement service for our own delegate
<br><br>

---
title: Module 7
subtitle: Changing GDB's Initialization

- DSF-GDB Launch steps
<br><br>
- Communicating directly with GDB
<br><br>
- FinalLaunchSequence
<br><br>
- Extending the FinalLaunchSequence
<br><br>
- DSF Exercise 9

---
title: Modifying GDB initialization

- DSF-GDB initializes GDB based on parameters of the launch config, e.g., 
    - Enable non-stop mode
    - Connect to a remote target
    - Open a core file
<br><br>
- In some situations, we may want to modify how GDB is initialized, e.g.,
    - Remove step that connects to remote target
    - Modify which .gdbinit file is read
    - Send a new command before connecting to target

---
title: DSF-GDB launch steps

- <code>GdbLaunchDelegate#launch()</code> called by platform
    - <code>FrameSpyLaunchDelegate#launch()</code> for our extension
- <code>ServicesLaunchSequence</code> triggered to start services
    - This uses the Services Factory we saw earlier
- <code>FinalLaunchSequence</code> triggered to initialize GDB

---
title: Adding a step at initialization

- **Goal**: Turn on GDB verbosity (debug printouts) from the beginning
    - Need to send GDB the MI command: <code>-gdb-set verbose on</code>
    - Need to send it before other commands sent to GDB
<br><br>
- **Exercises**:
    - Provide API in <code>FrameSpyService</code> to send new command
    - Extend DSF-GDB's initialization class: (<code>FinalLaunchSequence</code>)
    - Use extended initialization class instead of DSF-GDB's one

---
title: Communicating with GDB

- <code>ICommandControl#queueCommand()</code> used to send a command to GDB
<br><br>

<pre class="prettyprint" data-lang="java">
ICommandControl#queueCommand(
            new MIExecContinue(threadContext),
            new DataRequestMonitor<MIInfo>(getExecutor(), parentRm));
</pre>

<pre class="prettyprint" data-lang="java">
ICommandControl#queueCommand(
           new MIStackInfoDepth(threadContext),
           new DataRequestMonitor<MIStackInfoDepthInfo>(getExecutor(), parentRm) {
                @Override
                protected void handleSuccess() {
                    parentRm.setData(getData().getDepth());
                    parentRm.done();
                }
            });
</pre>

---
title: ICommandControl service

- <code>ICommandControl</code> also provides APIs for:
    - Getting notified of GDB commands status changes
        - *Queued*, *Sent*, *Removed*, *Done*
<br><br>
    - Getting notified of asynchronous GDB events.
        - <code>=breakpoint-created/deleted/modified</code>
        - <code>=memory-changed</code>
        - <code>\*stopped/\*running</code>
        - etc

---
title: Sending a command to GDB exercise

- Add a method to FrameSpyService that will send the command "-gdb-set verbose on"
    - Reset to **DSF9.1_START** or **DSF9.1_ADVANCED**
<br><br>
    - **Go!**

---
title: When to trigger this new command

- We want to enable debug logs
<br><br>
- We should send the command as early as possible

---
title: The ReflectionSequence class

<br><br>
- Used by <code>FinalLaunchSequence</code> to be extendable
<br><br>
- Born out of necessity
<br><br>

- Support grouping of steps.
    - In practice, only <code>GROUP_TOP_LEVEL</code> is used

---
title: The ReflectionSequence class (2)

- <code>getExecutionOrder()</code> returns array of steps using method **names**

<pre class="prettyprint" data-lang="java">
return new String[] { "stepGDBVersion",
                      "stepSetEnvironmentDirectory",
                      "stepSetBreakpointPending",
                    ...
</pre>

- <code>getExecutionOrder()</code> can be overridden to add/remove steps
<br><br>
- Steps are implemented by methods with specific **name** and tagged with <code>@Execute</code>
<br><br>

---
title: Extending GDB Initialization Sequence

- Have <code>FrameSpyFinalLaunchSequence</code> extend <code>FinalLaunchSequence</code>
<br><br>
- Add a new <code>stepSetVerbose()</code> to this initialization sequence
<br><br>
    - Reset to **DSF9.2_START** or **DSF9.2_ADVANCED**
<br><br>
    - **Go!**
---
title: Instantiating the new sequence

<br><br>
<br><br>
- As for previous steps, we need to instantiate our new sequence

---
title: Using new initialization Sequence

- Have <code>FrameSpyControlService</code> extend the existing ICommandControl service
<br><br>
- Override <code>getCompleteInitializationSequence()</code> to choose new launch sequence
<br><br>
    - Reset to **DSF9.3_ADVANCED**
<br><br>
    - **Go!**

---
title: Final Recap

- We've created a new view that used existing services
<br><br>
- We've created a new service that the new view can use
<br><br>
- We've created a replacement service for our own delegate
<br><br>
- We've modified GDB's initialization sequence

