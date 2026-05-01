---
name: "weekly-music-sheet-lyrics"
description: "Extract lyrics from Seraphim weekly music-sheet PDFs, find matching song links, OCR psalm/alleluia sections, and update the matching weekly Google Doc."
---

# Weekly Music Sheet Lyrics

## When to Use
- The user gives a Seraphim weekly music-sheet PDF and asks for song links or lyrics.
- The PDF includes a responsorial psalm (`Thanh Vinh`) and `Cau xuong Alleluia` that must be transcribed from the sheet.
- The target weekly folder contains a `.gdoc` Google Docs shortcut to update.

## Workflow
1. Work from `C:\Users\quyen\devs\Lyrics_extraction` unless the user gives another folder.
2. Inspect nearby weekly folders first to match the expected output format. Prior docs usually use:
   - one numbered heading per song or psalm,
   - bold headings,
   - plain lyric paragraphs,
   - no visible source-link list.
3. Render the PDF pages to images before relying on extraction. The PDF text layer may contain music-font glyphs or broken encodings.
   - Prefer `pymupdf`/`fitz` for rendering if Poppler is unavailable.
   - Save temporary renders under `tmp/pdfs/`.
4. Identify the four non-psalm songs from titles and visible composer names.
5. Search the web with this exact pattern whenever possible:
   - `"Name of the song" "Author" "Thanh Ca Viet Nam"`
   - Also try Vietnamese spelling variants such as `Thanh Ca Viet Nam`, accents removed, and first lyric line when the title is ambiguous.
6. For the `Thanh Vinh` dap ca and `Alleluia`, use OCR plus visual review:
   - OCR can propose text, but manually compare every line against the rendered sheet.
   - Preserve Vietnamese diacritics and liturgical wording.
   - Treat psalm/alleluia text from the PDF as the source of truth when web search is weak.
7. Update the weekly Google Doc, not the local `.gdoc` shortcut file.
   - `.gdoc` files on Google Drive for desktop may list but fail to read with `Incorrect function`.
   - Use the Google Drive/Docs connector to find the recent document by title/date, read it, then apply a Docs `batchUpdate`.
   - Preserve the prior weekly doc format unless the user asks for a different layout.
8. If adding source links, attach them as hyperlinks on the four non-psalm song heading lines instead of adding visible source clutter, unless the user asks for visible links.
9. Read the Google Doc back after editing to verify all sections, paragraph order, accents, and headings.

## Quality Checklist
- The target folder and PDF path were confirmed.
- The PDF was rendered or otherwise visually inspected.
- Four non-psalm song links were searched using the requested query pattern.
- The psalm response and alleluia were OCRed and manually corrected against the page image.
- The Google Doc was updated and read back successfully.
- Any uncertain author/source mismatch is called out in the final response.

## Useful Commands
```powershell
New-Item -ItemType Directory -Force -Path 'tmp\pdfs' | Out-Null
@'
from pathlib import Path
import fitz

pdf = Path(r'PDF_PATH_HERE')
out = Path('tmp/pdfs')
doc = fitz.open(str(pdf))
for i, page in enumerate(doc, start=1):
    pix = page.get_pixmap(matrix=fitz.Matrix(2.0, 2.0), alpha=False)
    pix.save(str(out / f'page-{i:02d}.png'))
'@ | python -
```

```powershell
$env:PYTHONIOENCODING='utf-8'
@'
from pathlib import Path
import easyocr

reader = easyocr.Reader(['vi', 'en'], gpu=False, verbose=False)
for page in [2, 3]:
    result = reader.readtext(f'tmp/pdfs/page-{page:02d}.png', detail=0, paragraph=False)
    Path(f'tmp/pdfs/easyocr-page-{page:02d}.txt').write_text('\n'.join(result), encoding='utf-8')
'@ | python -
```
