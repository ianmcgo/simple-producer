# Simple Producer: design notes

A single-file web app for laying out RPG supplement pages. Open `index.html` in a browser, or host it on GitHub Pages. Created by Ian McGonigal.

## Menus

- Page: Undo and Redo, Add pages (any number), Delete pages (type a list like `1-3` or `2,5,7`), Page background, Draw, Add header and Add footer, a closed fold-out named "Show and tools" holding Show page numbers, Snap to grid, Spell check and Show page indicator, Night mode, zoom slider with Fit width and Fit height, Save as JSON, Open JSON, Print, Print all as PDF, Delete all boxes (press twice to confirm; Undo brings them back).
- Insert: Text block, Image box (an empty drop zone), Table (3 x 3).
- Edit: appears while a box is selected. Its sections depend on the box type.
- The buttons are centered on the top bar. "Page 1 of 2" is a small badge at the bottom right of the screen and can be hidden in the Page menu.
- The background behind the pages is a field of slowly falling stars. Scrolling or zooming the pages does not move them. It stops if the system asks for reduced motion and never prints.

Drop-down panels never show a scrollbar. If one is taller than the window it still scrolls with the wheel or a swipe.

## Every box

- Top tab: drag "Move" to reposition. The small x at its right end deletes the box.
- Round handle at the bottom right resizes (works on rotated boxes too).
- Round handle on the left side rotates the box; it snaps to 5 degrees (hold Shift to snap when Snap is off). When a box is rotated, the Edit menu shows the angle and a "Reset rotation" button.
- Drag a box onto another page and let go to move it there. The page under the pointer receives it. The desk auto-scrolls near its top and bottom edges, and zooming out helps. Header and footer boxes stay put.
- Edit has Cut, Copy and Paste. Ctrl or Cmd + C, X and V work on boxes whenever you are not selecting text. A click in a text box with nothing selected copies the whole box. The Edit menu's Paste button also pastes an image you copied elsewhere (for example with right-click, Copy Image in another tab or app): it fills the selected empty image box, or makes a new one. If the browser refuses clipboard access it says so, and Ctrl or Cmd + V still works.
- "Place on page" is a fold-out just above Delete block. It holds a 3 x 3 grid: top, middle or bottom, and left, center or right, with a 0.5 in margin.
- Delete key removes the selected box when you are not typing.

## Text formatting

- In the Edit menu the text tools sit in a fold-out named Text editing. For tables it starts closed, along with a second fold-out named Cell editing for the row, column, merge and resize tools; for other boxes Text editing starts open.
- Select some words and bold, italic, underline, font, size, text color, alignment, numbered or bulleted lines, and links apply to just those words. With nothing selected they apply to the whole box (to the selected cells in a table).
- Links keep the text color and underline. Ctrl or Cmd + click opens a link.
- Night mode turns pages black, boxes and tables dark grey, and forces all text white, whatever color it was given (drawings keep their colors). Turning it off restores the original text colors.
- Boxed text boxes show Border thickness and Border color.
- Spell check toggles the browser's spell checker for every box.

## Page background and imported files

- Page > Page background sets the paper color and its opacity. A checkbox applies it to all pages or just the current one. Reset returns to the default paper.
- Import PDF or Word turns each page of the file into a locked background picture, starting on the current page and adding pages as needed (up to 20 pages per import). You cannot edit that text. Build over it with normal boxes, and use Imported opacity to fade it. Remove clears it from the current page.
- PDFs are drawn with pdf.js. Word .docx files are converted to HTML with mammoth.js and drawn with html2canvas, so the result is a simplified look of the document, not an exact copy. Word pages are split between paragraphs, tables and images; an item taller than a page is cut off. Old .doc files are not supported.
- The libraries load from cdnjs the first time you import, so that step needs an internet connection.
- Pages store the file as pictures in `pages[i].bg = {src, op}` and the paper color as `pages[i].color` and `pages[i].op`.

## Header and footer

Page > Add header or Add footer adds one box. In its Edit menu, type the pages it shows on: `all`, `1-2`, `3,5,7`, `odd`, `even`. Its text is shared, so editing it on one page changes every page it appears on.

## Tables

- Drag across cells to select several. Select row, Select column, Select all, and Select range (tap a cell, then tap another to extend) are in Edit.
- Add or delete rows and columns, merge and split cells, vertical alignment, cell color, text alignment, text formatting on the selected cells.
- Autofit narrows the selected columns to their text and lets rows shrink.
- Resize cells on shows handles on every cell edge; drag to change one column width or row height.
- Limits: up to 8 columns and 30 rows. Adding or deleting a row or column through a merged cell splits that merge.

## Images

An empty image box fills three ways: click it to choose a file, drop an image file onto it, or paste an image (Ctrl or Cmd + V) while it is selected. Dropping an image onto bare page space makes a new image box there, and pasting with a filled box selected makes a new box.

Replace, fit (inside, crop, stretch), Opacity, Flip left-right, Flip top-bottom, Border width and Border color, background color, and "Text under image", a caption you edit in place. Selecting caption text brings up the text formatting options. "Reverse" is provided as the two flips.

## Drawing

Page > Draw opens a pen bar with color, opacity and thickness. Draw on one page with a mouse, finger or pen. Undo stroke, Clear, Cancel, or Done. Done turns the strokes into a box you can move, resize, rotate, copy and delete. It is stored as vector strokes, so it stays sharp when resized.

## Saving your work

Page > Save as JSON downloads everything (pages, boxes, images, drawings, backgrounds, headers and footers, and the page settings) as `simple-producer-DATE.json`. Page > Open JSON loads one back and replaces what is on screen; Undo brings the previous work back. Opening a file checks and cleans it first: unknown box types are dropped, scripts and event handlers are stripped from text, and only png, jpeg, gif or webp data images are kept. Keep the files somewhere safe, since the browser's own saved copy is lost if you clear site data.

## Printing

Print and Print all as PDF both open the browser print dialog. Choose Save as PDF as the destination, margins None, and background graphics on. Pages are Letter size, one per sheet.

## Data model

State is saved in localStorage under `rpg-supplement-writer-v1`:

```
{ numbers, snap, bw, spell, where, running: [block], pages: [ { blocks: [block] } ] }
```

All blocks have `id, x, y, w, h` and optional `rot` (degrees), `color`, `fill`, `align`.

- Text: `font, size, boxed, html`, optional `bt` (border thickness), `bc` (border color).
- Image: `type:"image", src, fit`, optional `op` (0-100), `fx`, `fy`, `bd`, `bc`, `capOn`, `cap` (caption HTML).
- Table: `type:"table", head, rows, cw, rh`. Each cell is `{h, fill, align, valign, cs, rs}`; `cs` and `rs` are merge spans. `cw` holds column widths and `rh` row heights in px.
- Drawing: `type:"draw", dw, dh, strokes:[{c, o, w, p:[[x,y]]}]`.
- Header and footer (in `running`): text block plus `run:"header"|"footer"` and `pages` (the page list text).

Older saved files are upgraded on load: the old gaming sample text is swapped for the new neutral filler (other text is untouched) and table cells become objects.

## Decisions

- Pixel units are CSS px at 96 per inch; the page is 816 x 1056.
- Zoom uses the CSS `zoom` property. Drag math divides by zoom.
- The sample text is original neutral filler, not excerpts from novels.
- Undo keeps 60 states and is lost on reload.
- Text formatting uses the browser's `execCommand`, then converts `<font>` tags into styled spans.

## Known limits

- Work lives in this browser's localStorage. There is no file save or open yet.
- Text that grows past the page bottom is clipped, not flowed to the next page.
- Images are shrunk to 1400 px on the long side and stored in the saved data, so a few large images can fill the roughly 5 MB of browser storage.
- A drawing belongs to one page. Cut and paste of boxes uses an in-app clipboard (it does not leave the page).
- Cross-page drag preview shows a faint copy of the box; very small phone screens may find it easier to zoom out first.
