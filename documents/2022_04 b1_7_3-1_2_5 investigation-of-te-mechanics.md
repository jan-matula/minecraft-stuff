# b1.7.3-1.2.5: Investigation of TE mechanics

(April 2022, by Jan & Spheres a.k.a. v3rtices)

---

## Exposition of relevant variables

1. __The chunk TE map__: A per-chunk hash map from chunk block positions to TEs. The game uses this to look up the TE of a specific block.

2. __The loaded TE list__: A per-world list of TEs. TEs in this list will get updated/ticked in the TE phase.

3. __postpone-TEs flag__: A flag that gets set during the TE phase to make the game add TEs to the postponed TE list and prevent `java.util.ConcurrentModificationException`. See _TE postponing_ later in this document.

4. __The postponed TE list__ a.k.a. added TE list: A list of tile entities added during the TE phase. See _TE postponing_ later in this document.

Following are relevant properties of TEs:

1. __TE world object__: A reference to the world object. This is a TE's access to the world. (`null` by default.)

2. __TE coordinates__: TEs keep their coordinates, so even if when they are not in the chunk TE map and the game cannot look them up by block, as long as they are loaded they are still somewhat weakly linked to a block. (Coordinates are `0 0 0` by default.)

3. __TE-invalid flag__: This flag is `false` (for TE is valid) by default. It gets set when the TE is getting removed. Invalid TEs could be said to be _dead_ in a way.

---

## The TE phase

The TE phase can itself be subdivided into two separate sub-phases:

1. ticking TEs,
2. adding postponed TEs

For the duration of the entire first sub-phase, the _postpone-TEs_ flag is set. See _TE postponing_ later in this document. The game iterates through the loaded TE list. Valid TEs are ticked. Invalid TEs are removed from the loaded TE list and the chunk TE map via the remove-chunk-block-TE method.

In the second sub-phase, the game iterates through the entire postponed TE list. It adds all valid TEs from the list to the loaded TE list and then to the chunk TE map via set-chunk-block-TE. The chunk TE map coordinates are based on the TE coordinates that the TE itself keeps. Possible set-chunk-block-TE failures do not affect the loaded TE list – the TE can end up only being added to the loaded TE list.

---

## Chunk TE interface

The chunk TE interface only manipulates the chunk TE map directly.

1. __Get-chunk-block-TE__: Gets the TE from the chunk TE map (chunk TE map-get).

    If there is no TE for the block in the chunk TE map (TE is `null`) and the block is a container, the game attempts to fix the container. In b1.7.3, it does this by executing the block's on-added procedure. This can emit block updates in case of some blocks. In b1.8+, the game uses the containers get-TE method (returns the default TE for the block) and set-block-TE (described further below). After attempting to fix the TE, the game will get the TE from the chunk TE map again and return whatever result it got now.

    If there is a TE for the block, but it is invalid, the game removes it directly from the chunk TE map and returns `null`.

2. __Set-chunk-block-TE__: The game sets the TEs world object and coordinates. Then, if the block is a container, it will also validate the TE and put it in the chunk TE map (chunk TE map-put).

3. __Remove-chunk-block-TE__: Removes the TE from the chunk TE map (chunk TE map-remove); invalidates it.

## World TE interface & TE postponing

1. __Get-block-TE__: Gets the TE via corresponding chunk's get-chunk-block-TE.

    In 1.0+, if the TE is `null`, the game will iterate the postponed TE list to find a valid (not invalidated) TE with matching coordinates.

2. __Set-block-TE__: Does not do anything with invalid TEs (and in b1.8+ also `null`). In b1.7.3 `null` argument would throw `java.lang.NullPointerException` (crashing the game) when TE validity check is attempted.

    If TE postponing is off (the game is not in the TE phase), the game adds the TE to the loaded TE list (loaded TE list-add). Then it sets the TE via set-chunk-block-TE.
    
    If TE postponing is on (the game is in the TE phase), the game sets the TEs coordinates (— __it does not set the world object! The TE world object remains `null`__ —) and adds the TE to the postponed TE list (postponed TE list-add).

3. __Remove-block-TE__: Gets the TE via get-block-TE. Does not do anything for a `null` TE.

    If TE postponing is off, the game removes the TE from the loaded TE list (loaded TE list-remove). Then it uses remove-chunk-block-TE.
    
    If TE postponing is on, the game simply invalidates the TE. (Keep in mind that invalid TEs are removed during the TE phase.)

---

## Some notes and exclamations

1. In beta, TEs added during the TE phase (with TE postponing necessarily on) will not be detected by get-block-TE until the end of that TE phase. This is the most important takeaway for TE postponing in beta.

2. In 1.0+, TEs added during the TE phase _will_ be detected by get-block-TE; however their world object will remain `null` until the end of that TE phase. This is an issue for TE phase 0-ticking.

3. In 1.0+, postponed TEs _will_ be detected by get-block-TE and this is regardless of whether there is a container block at their coordinates.

4. In beta, get-chunk-block-TE — and therefore also get-block-TE and remove-block-TE — can cause block updates. This is if the block is a container and chunk TE map-get returns `null`. The game will execute the on-added procedure, which in the case of furnaces, dispensers, chests, and end portal blocks can cause block updates.

5. Invalid TEs will get directly removed from chunk TE map (chunk TE map-remove) by get-chunk-block-tile-entity.
