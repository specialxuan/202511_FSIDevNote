       SHORT TUTORIAL FOR USING CalculiX GraphiX (cgx) AS PREPROCESSOR

Conventions in this tutorial:

K: keyboard command
LMP: press left mouse button
LMR: release left mouse button
LMPR: press and release the left mouse button
MM: press middle mouse button
RM: press right mouse button
M: move mouse (no pressing or releasing)
V: visible result
E: press enter
** : comment

**start**
K: start cgx (e.g. with "cgx -b cube.fbd", 
   if you named the executable "cgx", put it in the directory
   "/usr/local/bin" and would like to create a model in file "cube.fbd")
V: a drawing window is created with on top "CalculiX GraphiX No. 1", on the bottom
   "cube.fbd", a drawing area within which there is a coordinate system
   x-y-z and a mouse pointer.

**

M: move mouse into newly created window
K: type "pnt p1 0 0 0" (without the ""), press enter (E)

** keyboard commands are accepted only if the mouse pointer is inside the 
   drawing window (can but does not have to be inside the drawing area) **

V: the text "pnt p1 0 0 0" appears in the window you started cgx in.

K: type "plot p all" E
M:
V: a blue dot appears in the drawing window

**

K: type "pnt p2 1 0 0", E
M: move mouse in the drawing window to the left of the drawing area or below 
   the drawing area (this area is also called the menu area)
LMP
V: a grey menu appears, on top "viewing", on the bottom "QUIT".
M: move the mouse down to the item "frame"
LMR
V: two blue dots representing the points appear in the drawing area.

**

K: type "plot pa all" E
V: blue labels  "p1" and "p2" appear next to the blue points

**

K: type "plot pa all r" E
V: the dots and labels are red now.

** g=green, y=yellow, k=black, b=blue m=magenta**

** if you mistype any letter, move back with the
   back key and retype the wrong letters; you can get the
   previous command back by using the arrow up key **

K: type "qlin" E
M: move the mouse inside the drawing area
K: type "r"
M: move the mouse pointer a little bit to the right and down
K: type "r"
M: move the mouse a bit
V: the mouse pointer changes into a rectangular pointer, the size
   of which is determined by the positions at which you typed the r's: the
   first "r" denoted the upper left corner, the second "r" the lower right 
   corner

** notice that for some commands, no enter (E) is needed after typing the
   command. This is, for instance, the case for "r" above **

M: move the new mouse pointer onto the dot symbolizing p1
K: press "b" (for begin)
V: in the typing window the point you marked appears, unless you missed
   the point and a beep is audible; in the viewing window the selected point
   gets a pink color, unless you missed it.
M: move the pointer onto the dot symbolizing p2
K: press "g" (for generate)
V: in the typing window the point you marked appears, and a description of
   the line you created

** to create a circle, mark the first point with "b", then the center point
   with "c" and finally the end point with "g" **

K: type "q" (to exit the qlin command)
V: the text "done" appears in the typing window
M
V: the mouse pointer returns to its original form

** if you do not quit, you can continue to draw lines using repeatedly
   "g" to create connected line segments, or interrupting with "b" to
   start a line from a new starting point; with "l" an existing line can be
   picked and redefined using "b", "c" and "g" **

K: type "plus la all" E
V: the newly created line appears together with its label.

** if you type "plus l all" E, no label appears **
** if you type "plot l all" the drawing starts from scratch and only the
   line appears (the points and point labels disappear) **

K: type "qadd se1" E
K: type "a"
V: after a fraction of a second the text "mode:a" appears in the typing
   window. Always wait for this text before proceeding.

** now create a window encompassing the line **

M: move the mouse pointer to the left upper corner of an imaginary rectangle
   encompassing the line
T: type "r"
M: move the mouse pointer to the right lower corner of an imaginary rectangle
   encompassing the line
T: type "r"
M
V: a rectangle appears with the size you just defined

** if nothing happens, LMPR in the menu area and returning the mouse into
   the drawing area usually helps **

M: move the rectangle such that the line is inside it

** if the rectangle proves to small, restart the marking of the corners with 
   "r" **

K: type "l" (for lines)
V: in the text window the lines "caught" by your rectangle are listed (here 
   only L1); in the drawing area the line is shown pink
K: type "q" (to quit the qadd command)

** by the previous command you created a set with name "se1" and containing
   one line with name "L1" **

** notice that the line name was given automatically. Names provided by the
   program always use CAPITAL letters **

K: type "plot la se1" E
V: the lines and their labels contained in set se1 appear in the drawing area,
   in this case just one line

K: type "plot la all" E
V: nothing really changes

** the set "all" contains everything and always exists **

K: type "swep se1 se2 tra 0 1 0" E
M
V: four lines and their labels appear now in the drawing window. 

** by the swep command you swept the line using a translation with direction
   vector 0 1 0; thereby three new lines (L002,L003 and L004) and a surface 
   A001 were created **

K: type "plus sa all" E
V: the surface A001 and its label appear in magenta

K: type "plus la se2 g" E
V: the line in set se2 (defined by the swep command) and its label appear
   in green

** the swep command takes two set names: the first is the set being swept,
   the second is the target set in which the swept entities are stored in
   their final position **

K: type "prnt se" E
V: in the typing window the names of all existing sets appear: se1, se2 and
   all

** create set se3 which contains surface S1 **

K: type "qadd se3" E
K: type "a"
M: move the mouse pointer to the upper left position of a rectangle containing
   S1
K: type "r"
M: lower right position of rectangle
K: type "r"
M
V: rectangle appears
M: move such that the surface (or at least its label) is in the rectangle
K: type "s" (for surface)
V: the surfaces caught appear in the typing window and acquire the color pink
   in the drawing area
K: type "q" (for quit)
M: the mouse pointer takes its original form

** if you omit  "a" in the above procedure, maximum one surface can
   be picked with each "s" command; the "a" command ensures that by
   typing "s", all surfaces whose labels lie within the pointing 
   rectangle are put into the set; no matter whether you use "a" or
   not, the "s" (or equivalently for picking other entities:
   "p"=points, "l"=lines, "b"=bodies, "n"=nodes, "e"=elements) command
   can be repeated after moving the mouse, in order to pick other 
   entities **

LMP
M
V: the surface in the drawing window rotates
LMR
MMP
M
V: the surface increases or decreases in size
MMR
RMP
M
V: the surface is translated
RMR

** the three mouse buttons are used to rotate the structure in the 
   drawing window, to make it smaller or bigger, or to translate it **

K: type "swep se3 se4 tra 0 0 1" E
M
V: a total of six surfaces and 12 lines are drawn in the drawing window:
   your created a cube by translating surface A001.

K: type "plot ld all" E
V: all lines appear with their division

** each line has a default division of 4. This means that four linear elements
   or two quadratic elements can be created along each line **

K: type "elty all he8" E
K: type "mesh all" E
K: type "plot m all" E
V: the cube is smoothly coloured with the colour green
M: move the pointer into the menu area
LMP
M: select the menu viewing and the submenu "LINES"
LMR
V: the surface colouring disappears and green lines appear marking 64
   elements

** 8-node brick elements are he8-elements; other element types are:
   tr3: 3-node 2-D elements (linear)
   qu4: 4-node 2-D elements (linear)
   qu8: 8-node 2-D elements (quadratic)
   he20: 20-node brick elements (quadratic)
   tr3, qu4 and qu8 elements are used to mesh surfaces, he8 and he20
   to mesh bodies **

K: type "plot e all" E
V: nothing really changes
K: type "plot ea all" E
V: the elements AND their labels appear (in green)
K: type "plus na all" E
V: the nodes and their labels appear (in red)

** from the above you can tell that a body must have been created.
   Indeed: **

K: type "plot ba all" E
V: the body B1 appears with its label

** the black structure edges can be removed by selecting "viewing -
   toggle model edges" in the menu (move pointer into menu area) **

K: type "plot ld all" E
V: lines and divisions appear

K: type "qdiv" E
M: move the mouse pointer onto one of the lines
K: type "2"
M
V: the division of the line changed from 4 to 2
M: move the pointer onto another line
K: type " 12" (note the space before the 12!)
M
V: the divison changed to 12
K: type "a" 

** wait till "mode:a" appears in the typing window, now create a rectangle
   by marking its corners with "r", and move it so that all lines are within
   the rectangle **

K: type "4"
V: all lines have division 4 again
K: type "q" (to leave qdiv)

K: type "del me all" E

** mesh is deleted **

K: type "elty all he20" E

** new element typ is assigned to set "all" (which contains all entities) **

K: type "mesh all" E
K: type "plus e all" E
V: now 8 quadratic elements were created (two along each line, since the
   division is 4)

** now select the "orientation, +y view" in the menu and move the mouse pointer back into the
   drawing area **

V: the structure is looked at along the y-axis in +y direction

K: type "plus n all" E

** now pick all nodes with z-coordinate 0 and put them into a set with
   name fix **

K: type "qadd fix" E
K: type "a"
M: to left upper corner of rectangle
K: type "r"
M: to right lower corner of rectangle
K: type "r"
M: move rectangle such that all nodes with z=0 lie within the rectangle
K: type "n"

** if the rectangle was too small, you can move it and repeat the command "n",
   nodes picked by consecutive "n" commands are all added to the set fix **

K: type "q"

K: type "plus n fix b"
V: all nodes in set fix are blue coloured

** repeat the same for all faces with z=1 and put them in set load: **

K: type "plot f all" E

** now select "viewing - FILL" and subsequently "viewing - Toggle Element
   Edges" from the menu; all faces are shown yellow; faces are external sides
   of elements and only exist after meshing **

K: type "qadd load" E
K: type "a"
M: to left upper corner of rectangle
K: type "r"
M: to right lower corner of rectangle
K: type "r"
M: move rectangle such that all faces with z=1 lie within the rectangle
K: type "f"
V: all faces caught get the color pink. It is impossible to pick only the
   faces on top, you will have caught some of the side as well. We will take
   care of this next.
K: type "q"
K: type "plot f load"
V: this shows all faces belonging to set "load"

** now we want to get rid of the faces not on top of the cube and belonging to
   set "load" **

K: type "qrem load" E

** this command removes entities from set "load" **

K: type "a"
M: to left upper corner of rectangle
K: type "r"
M: to right lower corner of rectangle
K: type "r"
M: move the rectangle such that all faces not on top of the cube lie within
   the rectangle
K: type "f"
V: all faces caught get the color pink. They are removed from set "load"
K: type "q"

** in a similar way one can also pick elements **

K: type "send all fbd"
V: some text in the typing window, ending with "ready"

** by this action a file "all.fbd" was created containing the geometric
   entities of the model (points, lines, surfaces, bodies, sets.. but no
   nodes or elements); by rename this file into "cube.fbd", the model
   can be reloaded into another cgx-session by the same command
   you started this session with)

K: type "send all frd"
V: some text in the typing window, ending with "ready"

** by this action a file "all.frd" was created containing the nodes and
   elements of the model. It can be reopened in another session by the
   command cgx all.frd, similar to the command
   opening this session, but without "-b". However, no geometric entities
   are contained in this file **

K: type "send all abq"
V: some text in the typing window, ending with "ready"

** by this action a file "all.msh" was created containing the nodes and
   elements in ABAQUS format; in addition an empty file "all.temp" was
   generated. If temperatures are defined this file is not empty **

K: type "send fix abq spc 23"
V: some text in the typing window, ending with "ready"

** by this action a file "fix.bou" was created fixing the nodes in
   set fix in direction 2 and 3, in ABAQUS format. This file can 
   be included into an ABAQUS deck. To fix just one degree of freedom,
   e.g. 1 just type "send fix abq spc 1". **

K: type "send load abq pres 10."
V: some text in the typing window, ending with "ready"

** by this action a file "load.dlo" was created containing the ABAQUS cards
   for a distributed pressure of magnitude 10. on all element sides to 
   which the nodes in set load belong **

K: type "send fix abq nam"
V: some text in the typing window, ending with "ready"

** by this action a file "fix.nam" was created containing the node labels
   collected into a set in ABAQUS convention. This can be useful to apply 
   forces onto a node set, or assign a different material to an element
   subset... **

K: type "help"
V: a help menu appears in the typing window containing much more commands
   than the ones described here

** now select "-QUIT-" from the menu: you quit the session, and should find
   "all.fbd", "all.frd".....in your directory.

GOOD LUCK!