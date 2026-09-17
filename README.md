Graphical implementation of the 

- layered coding of [Barmpalias and Lewis-Pye (2019)](https://arxiv.org/abs/1710.02092)
- simplified by [Shen (2023)](https://arxiv.org/pdf/2304.04852))

which dynamically converts multi-branching to equivalent 2-branching trees.

Clicking above a node in the left tree creates a successor of length the grid-distance of the node from the click.

<p margin-top="400px" align="center"><img width="650"  src="./tree-steps.svg"></p>

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

Random generation always keeps `w < 1` and observes the configured structural rules: the root has at most one child, no node is placed one level above the root, parent-child grid distance is at most 3, long unbranched runs are avoided, and the applicable single-child span limits are maintained. The height grows automatically as needed, without a fixed level cap.

### Browse earlier stages

The **back (‹)** and **forward (›)** buttons move one successful node addition at a time through the construction history, including manual, Step, and loop additions. Both trees, their colors, the load, and the grid scale return to the selected stage. Navigation pauses a running loop. Buttons are disabled at the beginning or end of the history.

Adding a node while viewing an earlier stage replaces the later stages with the new continuation. The top **gif** button exports the sequence through the selected stage; navigating alone preserves the later stages so you can return with **forward**. **Reset** clears the entire history.

### Inspect node matches

Hover a colored L-node to see its L-node number and highlight its current R-node match in the R-tree panel. Hover a colored R-node to see its R-node number and the L-node for which that R-node was created; its corresponding L-node is highlighted in the L-tree panel.

### Export the current view

- The **download icon** before **svg** in the top export group downloads `L-R-trees.txt`, containing tree-style diagrams for the previous and current stages. The diagrams list branches depth-first, from left to right, using `├`, `└`, `─`, and `│` connectors; horizontal edge length is proportional to the nodes' grid-level difference. The R-tree export includes only colored nodes; when hidden encoding nodes sit between them, it connects a colored node directly to its nearest colored R-tree ancestor. Its displayed R-node numbers are reassigned consecutively from `0`, so hidden-node creation IDs never leave gaps, and each R-node label has the form `r(l)`, where `r` is that displayed R-node index and `l` is the L-node for which it was created.
- **svg** downloads the current two-tree construction area as `tree-builder.svg` and `tree-builder.html`. The HTML file embeds the exact same SVG markup directly in its body, ready to display in a browser or copy into another HTML page. Direct **SVG** and **HTML** download links also appear below the trees, so either file can be downloaded individually if an automatic download does not arrive. These links retain the latest exported snapshot until the next SVG export.
- The top **gif** button becomes enabled after the first successful L-node addition, whether made manually, with **Step**, or by **loop**. It records the initial roots and one frame after every successful addition since **Reset**. Press **gif** to download a looping `tree-steps.gif` and a looping `tree-steps.svg` containing the sequence up to the moment the button was pressed. The SVG animation keeps each recorded frame as vector graphics. Both exports include the control panel and both tree panels, with 0.6 seconds per frame. Export is also available during a loop; later additions appear in the next export.
- The **gif** beside **splice** records one splice and split cycle, then downloads `splice-split.gif` and its vector-based animated SVG equivalent `splice-split.svg`. These exports contain the R-tree transition area and use the same transition cadence as the browser view.

Starting, pausing, or resuming a loop preserves the recorded sequence. **Reset** clears it and disables the top **gif** button until another node is added.

### R-tree transition controls

- **splice** moves the visible R-node groups into their corresponding L-tree positions. While this layout is active, the same button is labeled **split**.
- **split** returns those groups to the encoded R-tree layout. The edge movement is animated so each transition can be inspected before the final layout is drawn.
- **width growth** expands or collapses the i-width comparison chart below the trees.

### R-tree generation algorithm

The R-tree is built incrementally, in the order L-nodes are added. Both trees start with a black root at grid level 0. Every L-node has a unique RGB color selected to remain visually distinct from existing L-node colors. Color identifies an L-node: its visible copies in R have exactly the same color and grid level. One L-node may have several R-copies; the node numbers in L and R are independent creation numbers.

When a new L-node `v` is added as a child of `x`, the algorithm proceeds as follows:

1. **Use the current match of `x`.** Each L-node has one current R-match: its newest visible R-copy. A new L-child `v` is attached to this match.
2. **Replace a saturated match.** If the current R-match `r` has no room for the new branch, split the incoming R-edge at the grid line immediately below `r` and add a new copy `r'` beside `r`. If that attachment point is also full, copy the required attachment path first. The copy has the same color and grid level as `r`, and becomes the current match of `x`.
3. **Add the new match.** Attach the new R-copy of `v` above the current match of `x`, preserving `v`'s color and grid level.
4. **Fill skipped grid levels.** Before a visible copy, insert one hidden R-node at every intermediate grid level. Hidden nodes have no colored circle and are not L-node matches.
5. **Update the layout.** Redistribute horizontal positions and rescale the shared vertical grid as needed. Existing R-parent relationships are preserved; the operation adds a new chain.

For example, if a new L-node is added under green L-node 4 and its current green R-match is full, a new green copy is created beside that match first. The new L-node's R-copy then attaches above the replacement copy; the replacement is the new current match for L-node 4.

Every R-node has at most two children. The **splice/split** controls change the display without changing this underlying R-tree structure.
