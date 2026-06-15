Fetch the song from pisnicky-akordy.cz and convert it to bard.md format for this songbook project.

The URL is: $ARGUMENTS

## Steps

1. Fetch the page using curl with a browser user agent:
   ```
   curl -s -A "Mozilla/5.0 (X11; Linux x86_64; rv:120.0) Gecko/20100101 Firefox/120.0" "<URL>"
   ```

2. Extract the raw HTML inside `<div id="songtext">` and parse it with Python:
   - Chord lines are `<el class='aline'>` elements containing `<span class='akord'>CHORD</span>` spans
   - Lyric lines are the plain text lines immediately after each chord line
   - Extract the song title from `<h1>` and artist from `<h2>`
   - Extract capo from the text if present (e.g. "Capo III." → `*Capo 3*`)

3. Align chords with lyrics character by character:
   - The chord line and lyric line share the same column positions (monospace)
   - For each chord, find its column position in the chord line
   - Find the character at that column in the lyric line — place the chord backtick there
   - Result: inline chord notation like `` `Am`Víno `C`máš a `G`marky`Am`tánku ``

4. Identify song structure from lyric line prefixes:
   - `*:` prefix → intro/outro repeated section → use `>` for first occurrence, `!>` for last
   - `R:` prefix → main refrain → use `>>` (second chorus)
   - `1.`, `2.`, etc. → numbered verse lines
   - Refrain repeats → use `!>>` to reference the `>>` block
   - Continuation lines (no prefix, indented) → bare lines under the previous section

5. Convert chord names to German notation (as set in bard.toml):
   - `Am` → `Ami`, `Em` → `Emi`, `Dm` → `Dmi`, `Hm` → `Hmi`
   - `B` → `B` (Bb in German notation), `H` stays `H`
   - All other chords stay as-is

6. Determine the output filename from the song title:
   - Lowercase, replace spaces with hyphens, strip diacritics or keep them
   - Match the convention of existing files in `songs/` (e.g. `batalion.md`, `blízko-little-big-hornu.md`)

7. Write the file to `songs/<filename>.md` with this structure:
   ```
   # Title
   ## Artist
   *Capo N*        ← only if capo is specified
   
   > (intro/outro lines if present)
   
   1. (verse 1 lines with inline chords)
   
   >> (refrain lines with inline chords)
   
   2. (verse 2 lines)
   
   !>>
   
   !>            ← only if there's an intro/outro
   ```

8. Run `bard make` to verify the file compiles without errors.

## Chord inline format rules (from existing songs in this project)

- Place chord directly before the syllable where it falls: `` `G`markytánku ``
- Mid-word splits are normal when the chord falls mid-word: `` `G`marky`Ami`tánku ``
- Hyphens with spaces indicate syllable-level timing, preserve them: `po- há- ní`
- Empty chord `` `` `` means a chord at end of line with no following text
- Only the first line of a chorus/verse block needs the `>` / `>>` / `1.` prefix

## Important notes

- The project uses `notation = "german"` in bard.toml
- The chorus label is `"R"` in bard.toml — the `>` block gets labeled "R" in output
- Look at existing songs in `songs/` for style reference, especially `batalion.md`
