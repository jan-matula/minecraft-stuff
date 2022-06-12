# b1.7.3-1.2.5: TE phase piston mechanics

(April 2022, by Jan & Spheres a.k.a. v3rtices)

---

## General timing

1. (TE phase) B36 takes 3 gt to finalize: pistons take 3 gt to properly finish extending; pistons take 3 gt to retract.

    ___\[tested in b1.7.3, b1.8, 1.2.5 works\]___

    This is because the B36 TE is postponed and added to loaded TE list only after the TE phase has ended. It will get ticked for the first time during the TE phase of the next tick. This is somewhat similar to pistons activated during the TE phase only extending 1 gt later in newer versions.

---

## Exposition of relevant b36 mechanics

1. __clear-b36-TE__: Finalizes the b36. This is not used when the b36 finalizes naturally, during its TE tick. It is used in the following contexts:

    - On retraction, pistons (normal or sticky) clear-b36-TE the piston head b36 TE.
    - On retraction, sticky pistons use clear-b36-TE in order to facilitate block dropping.
    - In b36 block on-removal, clear-b36-TE is used to remove the b36 TE.

    __Unless the TE world object is `null`__, the game will do remove-block-TE for its own coordinates (of course, this TE itself need not be the one in chunk TE map) and invalidate itself. If there is a b36 block in the TE's place, it will do set-block-and-metadata to turn into the moving block. __Without the b36 block, the set-block-and-metadata does not happen.__
    
    __If the TE world object in `null`, this is a no-op.__

2. __B36 on-removal__: Checks for the b36 TE. If it does find one, it does clear-b36-TE on it. Otherwise it uses the general container on-removal, which simply involves removing the block's TE regardless of its type.

It is worth exclaiming that:

1. Proper sticky piston 0-tick behaviour facilitating instant block dropping is not possible.
2. B36 blocks created during TE phase cannot remove their TE during that TE phase.

---

## Bugs / exploits

1. (TE phase) 0-tick piston (normal or sticky)
    &xrArr; Air with piston head b36 TE (only loaded TE list).
    
    ___\[tested in b1.7.3, b1.8, 1.2.5 works\]___
    
    The piston head b36 block is destroyed, but its b36 TE cannot.

    You could try placing a container where the air block is; however, any obvious way to do this will override the b36 TE with the container's TE in the chunk TE map anyways. On the other hand you can supply the b36 TE with a b36. The piston head B36 will be ticked before the b36 that is actually in the chunk TE map making it the one to actually finalize.

2. (TE phase) 0-tick sticky piston with a block to push
    &xrArr; No block dropping occurs; the sticky piston acts as a normal piston would. + Air with piston head b36 TE (only loaded TE list).

    ___\[tested in b1.7.3, b1.8 works, 1.0, 1.2.5 different behaviour\]___

3. (TE phase) 0-tick sticky piston with a block to push
    &xrArr; No block dropping occurs. + Double-headed piston.

    ___\[tested in b1.7.3, b1.8 different behaviour, 1.0, 1.2.5 works\]___

    Unlike in beta, the piston does find the b36 TE and does do clear-b36-TE. Finding an extending b36 TE with an orientation matching the piston to fall into block dropping behaviour. Not destroying the piston head is part of this behaviour.
    
    (The piston head b36 TE is, of course, also cleared with the same results.)

4. (TE phase) 0-tick sticky piston with a block to pull
    &xrArr; Double-headed piston.

    ___\[tested in b1.7.3, b1.8, 1.2.5 works\]___

    Here a b36 block is supplied to the piston b36 TE allowing it to finalize. The b36 block is the one of the pulled block.

5. (TE phase) 2 pistons facing into the same air block: 0-tick piston 1, extend piston 2
    &xrArr; Piston 1 becomes double-headed, piston 2 becomes headless.
    
    ___\[tested in b1.7.3, b1.8, 1.2.5 works\]___
