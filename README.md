Graphical implementation of the 

- layered coding of [Barmpalias and Lewis-Pye (2019)](https://arxiv.org/abs/1710.02092)
- simplified by [Shen (2023)](https://arxiv.org/pdf/2304.04852))

which dynamically converts multi-branching to equivalent 2-branching trees.

<p margin-top="400px" align="center"><img width="650"  src="./tree-steps.svg"></p>

Clicking above a node in the left tree creates a successor of length the grid-distance of the node from the click.

## Using the tree builder

Open `kcodeApp.html` in a modern web browser. The left panel is the editable **L-tree** (the multi-branching tree); the right panel is the generated **R-tree** (its binary-tree encoding). The R-tree is extended immediately after each L-tree addition. See [R-tree generation algorithm](#r-tree-generation-algorithm) for the conversion rules.

### Build a tree manually

1. The black node at the bottom of the L-tree is the root.
2. Click an unused grid position above an existing node. The closest node below the click becomes the new node's parent.
3. The vertical grid distance between the new node and its parent determines the new node's level. The colored node and its edge appear on the left, while the corresponding visible and hidden R-tree nodes are generated on the right.
4. Both trees and their grid lines scale down automatically as the trees grow. The tallest nodes stay on or below the second grid line from the top, leaving at least one empty grid line above the leaves for further additions. Click the top grid line or the 40-pixel space above it to extend the tree; this remains available as the grid spacing shrinks.

Manual additions are rejected if they would make the displayed load reach or exceed 100%. The pie chart beside **Load** shows the current load; its accessible label includes the whole percentage. **Reset** clears both trees, returns the load to 0%, and discards any recorded loop frames.

### Generate a random tree

- **Step** adds one random valid L-tree node and immediately updates the R-tree. New L-node colors are selected to maximize perceptual contrast with existing L-node colors.
- **loop** begins repeated Step actions every 0.6 seconds. Its label changes to **pause** while it is running; press it again to pause.
- **Reset** stops a loop (if one is active), clears the trees, and starts a fresh construction.

### Node matches and history nevigation

Hover a colored L-node to see its L-node number and highlight its current R-node match in the R-tree panel. Hover a colored R-node to see its R-node number and the L-node for which that R-node was created; its corresponding L-node is highlighted in the L-tree panel.

The **back (‹)** and **forward (›)** buttons move one successful node addition at a time through the construction history, including manual, Step, and loop additions. Both trees, their colors, the load, and the grid scale return to the selected stage. Navigation pauses a running loop. Buttons are disabled at the beginning or end of the history.

### Export the current view

- The **download icon** before **svg** in the top export group downloads a text file containing tree-style diagrams for the previous and current stages. 
- **svg** downloads the current two-tree construction area as an svg file (standalone and html-embeded)
- The top **gif** button downloads the transitions since the last reset as svg and gif files.
- The **gif** beside **splice** downloads gif/svg file with the plice/split transition which  transforms the R-tree into the L-tree.

### R-tree transition controls

- **splice** moves the visible R-node groups into their corresponding L-tree positions. 
- **split** returns those groups to the encoded R-tree layout. 
- **width growth** expands or collapses the i-width comparison chart below the trees.

### R-tree generation algorithm

The R-tree is built incrementally, in the order L-nodes are added. Both trees start with a black root at grid level 0. Every L-node has a unique RGB color selected to remain visually distinct from existing L-node colors. Color identifies an L-node: its visible copies in R have exactly the same color and grid level. One L-node may have several R-copies; the node numbers in L and R are independent creation numbers. When a new L-node `v` is added as a child of `x`, the algorithm proceeds as follows: 

1. **Use the current match of `x`.** Each L-node has one current R-match: its newest visible R-copy. A new L-child `v` is attached to this match.
2. **Replace a saturated match.** If the current R-match `r` has no room for the new branch, split the incoming R-edge at the grid line immediately below `r` and add a new copy `r'` beside `r`. If that attachment point is also full, copy the required attachment path first. The copy has the same color and grid level as `r`, and becomes the current match of `x`.
3. **Add the new match.** Attach the new R-copy of `v` above the current match of `x`, preserving `v`'s color and grid level.
4. **Fill skipped grid levels.** Before a visible copy, insert one hidden R-node at every intermediate grid level. Hidden nodes have no colored circle and are not L-node matches.
5. **Update the layout.** Redistribute horizontal positions and rescale the shared vertical grid as needed. 

For example, if a new L-node is added under green L-node 4 and its current green R-match is full, a new green copy is created beside that match first. The new L-node's R-copy then attaches above the replacement copy; the replacement is the new current match for L-node 4.
