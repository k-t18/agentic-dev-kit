# Screen list from Figma

Section 4 (Screen inventory) starts from the design, not from memory.

## When a Figma link is given

1. Call the Figma MCP `get_metadata` on the linked node. It returns the frame tree —
   names, node ids, sizes — without the heavy per-layer data.
2. Take the **direct child frames** of the linked section or page as candidate screens.
3. Classify each candidate:

   | Looks like | Record as |
   | --- | --- |
   | Full-size frame (page width) | `page` |
   | Small frame meant to sit over a page | `modal` |
   | Several frames with the same name | variants of one screen — ask what differs |
   | A frame with a generated name (`Frame 2147…`) | unknown — ask what it is |
   | A component or style frame (colours, typography, icons) | not a screen — leave out |

4. Build each frame's link from the file URL and its node id, so the row points at the
   exact frame.
5. Use `get_screenshot` only when a name does not tell you what the screen is. Do not
   pull `get_design_context` — the spec does not record styling.
6. **Show the list to the PM and ask them to confirm it** before the flow round: which
   frames are real screens, which are variants of the same screen, what order they come
   in, and whether any screen is missing from the design.

Frame names are the designer's labels, not the truth. Never infer a rule, an order, or a
step number from a name — ask.

## When no link is given, or the Figma MCP is not available

Say so, then ask the PM to list the screens and modals in order and paste a frame link
for each. A screen with no design is still listed, with `no design` in the Figma column
and an open question asking who will supply it.

## What goes in the inventory

One row per screen: its ID, a name the team will use, `page` or `modal`, the frame link,
and the screen or action it is reached from. Variants of one screen share one ID and are
listed in the notes column with what distinguishes each.
