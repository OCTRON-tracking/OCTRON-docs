# Cleaning tracks
Trackers aren't perfect. Every so often, the tracker you used during [analysis](analysing.md) will briefly lose an animal - it slips behind an object, or the tracker just "lets go" of it — and picks it back up under a brand new ID once it's visible again. The result: one real animal ends up scattered across two (or more) separate tracks. Or maybe you just got a handful of junk tracks - false detections that aren't worth keeping.

The **Prediction cleaner** is a second napari dock widget built exactly for this. It lets you stitch broken-up tracks back together, or throw out garbage in napari - and since nothing is ever actually deleted from disk, you can always undo your changes.

<video width="100%" muted controls style="display: block; margin-left: auto; margin-right: auto;">
  <source src="../assets/videos/cleaner_ui_demo.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>

## Opening the cleaner
The cleaner widget pops up automatically whenever you load a set of prediction results — either right after [analysing](analysing.md) a video (when you clicked on the checkbox "View result"), or by dragging a results folder onto the napari window. You can also open it any time from napari's **Plugins** menu in the menubar.

## Picking a track to fix
Use the **source track** dropdown to pick the track you want to work on. Every entry is colour-coded by label name and shows how much of the video that track actually covers (in percent). Tracks with low coverage are usually the fragmented ones most in need of fixing, so they float to the top of the list.

Once you've picked a source track, OCTRON looks for **candidates**: other tracks sharing the same label that don't overlap in time with your source, and that start or end reasonably close to it in both time and space. These show up in the table below, together with an estimate of how much extra coverage you'd gain by joining them (colour-coded from grey to green - greener is better).

Not sure exactly where the gap between two tracks is? Click the little ◀ ▶ arrows next to a candidate to jump the timeline straight to the frame where one track ends and the other begins.

## Previewing a join
Tick the checkbox next to a candidate you think belongs to your source track. OCTRON isolates both tracks in the viewer so you can inspect them without any distraction, and fades their colors into a gradient right around the point where they would connect, so the join is easy to spot at a glance:

<div style="display: flex; gap: 2rem; justify-content: center; align-items: flex-end; margin: 1.5rem 0; flex-wrap: wrap;">
  <figure style="margin: 0; text-align: center;">
    <img src="../assets/towards_join_point.png" alt="Color gradient fading from light blue to black" style="width: 100%; max-width: 320px; border-radius: 4px;"/>
    <figcaption><em>towards</em> the join point</figcaption>
  </figure>
  <figure style="margin: 0; text-align: center;">
    <img src="../assets/awayfrom_join_point.png" alt="Color gradient brightening from black to yellow" style="width: 100%; max-width: 320px; border-radius: 4px;"/>
    <figcaption><em>away from</em> the join point</figcaption>
  </figure>
</div>

The end of the earlier track fades from blue into black **towards** the join point, and the beginning of the later track picks up right there in black and brightens into yellow moving **away** from it. If the animal's position and movement line up nicely right where the colors meet, you've probably found a genuine match.

!!! note "What you see is exactly what's on disk"
    Tracks are shown raw here - no smoothing, no gap-filling. A jittery or gappy trajectory right at the join point is the actual recorded data, not a display artifact, so you can trust what you see when deciding whether two tracks belong together.

## Saving a join
Happy with your pick(s)? Hit **Save**. OCTRON immediately fuses your source track and every checked candidate into a single, new track ID - merging the underlying .csv files and, for segmentation projects, the mask data too.

## Deleting a track
Want to get rid of a track entirely - because it's junk, or a false detection? Just delete its layer in napari: select it in the layer list and click the 🗑️ icon (or select it in the viewer and hit delete). OCTRON picks up on the removal automatically and archives that track's data.

!!! tip "Quick delete"
    Selecting a source track also selects its layers (the track and, if present, its masks) in napari's layer list - so you can jump straight to deleting it if that's what you came here to do.

## Undoing changes
Nothing you do in the cleaner is destructive. Every fuse or delete moves the original files aside into an archive folder rather than removing them, so you can always change your mind:

- **Revert:** undoes only the most recent fuse or delete.
- **Reset:** undoes everything you've done in the cleaner this session, back to how your results looked before you started.

## Under the hood: how saves and undo actually work
Nothing is ever edited in place, and nothing is ever truly deleted. Every fuse or delete writes its result to disk first, and only then moves the original files it touched into a per-operation archive folder, together with a small JSON "manifest" describing exactly what happened. That manifest is the undo log.

- **Fusing (Save):** OCTRON reads every member track's `.csv`, concatenates the rows, re-sorts by frame index, and writes the result to a brand new `<label>_track_<new_id>.csv`. If your project has masks, a matching new mask array is created in the results' zarr store, copying over every annotated frame from each member track. Only once that new data has been written successfully does OCTRON touch the originals: each member's `.csv` is *moved* (not copied) into the archive folder as `<original_name>.csv.bak`, and each member's mask array (if it has one) is renamed in place with a `fused_orig_` prefix - keeping it invisible to OCTRON's normal track scanning while it stays right there in the zarr store, ready to be restored.
- **Deleting a track:** the same idea, just for a single track and without creating anything new. Its `.csv` moves into its own archive folder, and its mask array (if any) is renamed with a `deleted_orig_` prefix.
- **The archive folder:** everything lands in a `fused_originals/` folder inside your results folder, one subfolder per operation, named after a timestamp so operations always sort chronologically - `<timestamp>_id<new_track_id>` for a fuse, `<timestamp>_del<track_id>` for a delete. Each subfolder holds the archived `.csv.bak` file(s) plus a `manifest.json` recording what kind of operation it was, which track IDs were involved, and which mask arrays were renamed (and to what).
- **Revert** looks up the most recent archive subfolder, reads its manifest, and undoes it: the archived `.csv.bak` file(s) move back to their original names, any renamed mask arrays are renamed back, and - for a fuse - the new track's `.csv` and mask array that were created are deleted again. **Reset** simply calls Revert repeatedly until the archive is empty, walking all the way back to how your results looked before you opened the cleaner.

After every save, revert or reset, OCTRON doesn't try to patch what's currently shown in napari - it clears every layer and reloads everything straight from what's now on disk. This guarantees the viewer always matches the real state of your files, rather than risking the two drifting out of sync.

!!! info "Coming soon: splitting tracks"
    Right now the cleaner only helps with joining fragments or deleting junk. We're planning an analogous **splitter** tool for the opposite problem - when the tracker quietly hands the same ID to a different animal partway through a video - so keep an eye out for that in a future update.
