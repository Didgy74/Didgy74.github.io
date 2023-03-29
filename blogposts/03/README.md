
# Revisiting automatic relayout GUIs

# Purposes of this document
My goal for this text is to give myself a more formal understanding of the problem at hand. This also serves as an exercise in learning how to write. It will allow other people to gain some insight into how I solve problems. Finally, I expect find some joy in solving the problem and describing it clearly.

# Rough description of the solution I want
This is mostly an academic experiment, I don't really expect to write something pragmatic or something people will want to use. If it turns out usable, great. If not, idc. The purpose of my experiment within GUI is mostly to see where we can push the limits, how do we approach a holistic solution for it.

With that in mind, I'm looking for a pedantic, puristic solution. I want to focus on finding the *theoretically* best solution. I'll also be avoiding adding rules, exceptions to rules or additional data-state in the name of convenience.

The keyword here really is *pedantic*.

Additionally, I've already implemented many of these ideas in my C++ already, but I wanted to see if it was possible to adjust the ideas to work in idiomatic Rust. The current C++ interface for this solution is very easy to accidentally misuse, and the type system is not utilized as well as it could be to enforce correct usage.

# Terminology
**Widget**
Any piece of the user interface that wants to take up screenspace, and might also want to respond to some events. 

**Screenspace**
The physical space that the GUI is displayed on. Usually this means the amount of pixels in the 
window for which we are displaying. Note: The reality is a bit more complicated because we want to take DPI into account.

**Rect**
A shorthand for rectangle. In the solution, this means a rectangle defined by position and extents.

# The problem
The problem I'm trying to solve is how to automatically place GUI widgets on the screen, and have them correctly respond to events. For the vast majority of events, we can only start applying the event once we know how big every widget. Turns out, when you want your widget to automatically readjust itself to take up screenspace in the best possible manner, this ordeal becomes a bit more complicated.

Every content-aware widget has a few criteria on how big it needs to be in order to be displayed correctly. A window has no upper or lower bound on how many widgets it needs to display, yet a window always has a specific size at the time of an event. Because widgets can request an infinite amount of screenspace, yet screenspace itself is finite, we need fit the widgets and that includes compromise. We need to adjust the size of all widgets in order to make the best possible compromise, with respect to each widget.

I think it's best to start with an example. Let's create a very simple combination of Widgets. First I will describe it in words, then I will demonstrate how we would expect this to show up on screen. I will label each widget alphabetically.

### Not finished

### Demonstrate how layout-style are a sum of their children, and how this functions recursively

### Demonstrate how changes to a widgets content (or size) can affect widgets in different parts of the GUI

# My solution - Conceptually
## Assumptions
- Widgets always take up a rectangular shape on screen
	- They can still be partially transparent and conditionally pass through events, making it seem that they are not rectangular from user perspective.
- Widgets appear as axis-aligned planes, meaning they cannot span multiple values in the Z-axis.
- Each rectangular space will end up to be pixel-aligned after calculations are complete.
- All GUI surfaces (read: windows) are rectangular grids of pixels.
  - Additionally it has certain properties like DPI to determine how physically large the surface is.
## End of assumptions
For my solution, I look at all GUIs as an N-tree. Each widget is viewed as a node, and widgets may also point to one or more child widgets. From now on, I will be using the word "node" and "widget" interchangeably. 

I'm introducing a new container type that I will call the "RectCollection". The RectCollection is a short-lived data structure that essentially captures the entire GUI architecture in a moment of time, and stores the Rects of every relevant widget. It is reconstructed every time you need information regarding the sizes of widgets, such as when processing an event. It can be queried after the event, but it is effectively invalidated if anything is changed in the GUI architecture.

Processing an event requires three steps:
1. Measure / Gather
2. Distribute
3. Process event

This process will have to be repeated for every event. An important idea here is that at every stage, the information gathered for that widget is passed back to that widget at the later stage.

**Gathering**
This is a bottom-to-top algorithm.

During the gather stage we gather the SizeHint of each widget. At this point we have no idea which widgets will actually be displayed, so we just gather this for every single widget that can potentially be displayed. If it's impossible for the widget to be displayed this event, we can omit it. Parents will report their SizeHint as a "sum" of their children, how that sum is defined is specific to that Widget. This step will run in a bottom-up direction, starting with the children that are nested the most deeply into the hierarchy. 

**Distributing**
This is a top-to-bottom algorithm.

During the distribution stage we assign the actual Rects of each widget. During this stage, nodes can **only assign the Rects of their children, not itself**. This rule enforces that nodes can only rely on the screenspace that they are given by their parent, and not set an arbitrary size. At this point in time, we can start determining which widgets will be visible or not, and so some may be omitted.

At this point in time, we have assembled all of the data necessary to perform some action. We now know the position and sizes of every Widgets Rect, and also in which order they appear in the Z order.

**Run actual event**
This is a top-to-bottom algorithm.

**Unfinished:** ... Do stuff, send data down, widgets use return value to tell you if they did anything meaningful with the event...



# Introducing temporary data
Now, there's a performance issue with this solution. Turns out that many widgets, particularly content-aware ones need to do some complex calculations to figure out their SizeHint. Additionally, some of those need to do similar complex calculations for handling events. Let's take an example:

We intend to do a rendering event. When we are done solving the layout, we want to render.  In our current layout, there is a lot of Text widgets, so we're gonna have to render a lot of text. We will be focusing on only one such text widget, and examine what it does throughout our process.

First it will be asked to figure out its SizeHint. It's content-aware, so it needs to check out how much text it has and how much space that text wants to take up. The algorithm goes through all of the text, adds up positions and extents and figures that out. When it's done, it discards the information containing positions and extents of each glyph, and it is left with only the outer bounds. Turn that into your SizeHint and we're done with this stage.

Later on, it will be asked to display that very same text. To do this, it will have to figure out what exactly the text is, and again loop over every glyph once more and find out where they are so it can send it to the renderer. We just did double the amount of work necessary.

Now, looking at just this one scenario, it's no big deal. But in my tests, it does actually become a problem when having many text widgets with each one having much text.

Example over. Here is my proposed solution:

Each node can define their own type of custom data, which will be handed back to them later in each stage.

Question: Is customdata immutable when handed back to Widget? I think no

# The focused Widget
In every GUI, you need to be able to give a Widget *focus*. Focused widgets listen to specific events, such as "Activate" or "Read text with system screen reader". Only one Widget per hierarchy can be in focus at any given time.

In my solution, the root node will have to be able to hold an optional reference to such a Widget, and track it if it moves to another location in the hierarchy. It will also need to be able to detect that it has been removed from the hierarchy altogether.

# Benefits
Multi-threaded, immutability correctness, can guarantee the state does not change during certain operations

# Disadvantages
Less intuitive to use
Long parameter lists

# Rationale (Read: copium)
So the solution presented here might seem to be overly complicated at a glance, and you'd likely be right in thinking so. But even so I'm gonna try to defend myself, so here's a few criticisms I could think of that I would like to respond to.

**Why not just store Rects and temp-data inside the widget itself?**
This is a good question, and this is what most GUI solutions do. It even has the added benefit of being able to query a widget, at any given, time what size it is or where it is. Quite convenient. The reason I'm not doing it this way is because of the fact content changes can dramatically impact not just the size of a single widget, but the GUIs layout as a whole. From what I can tell you have only two options for implementing this:

You only update the value at specific timepoints, such as during event processing. In this case, the size value will be blatantly incorrect most of the time in a dynamic GUI, even if you modified a different widget, and it would be quite convoluted to determine when this value is correct. This becomes even more complex in a system that supports reactive Widgets, the content doesn't even exist in the widget itself and is pulled from an external data source. How would you determine when this external data source changes?

The alternative is to trigger complete relayouting at every possible change, which means more state and more hidden operations happening under the hood.

Both these alternatives are directly opposed to the philosophy I want to apply when designing my solution, and my current solution is the best compromise I could find for now.

**Since the SizeHint of a Widget is a pure function with no side-effects, you can just query it everytime, why bother storing it?**
The TL;DR here is that it is possible, but introduces performance problems.
