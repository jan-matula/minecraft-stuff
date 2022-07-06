# 1.7.2: Falling block duplication

(July 2022)

Comment by Rubel on [Mxi10's video](https://www.youtube.com/watch?v=K3VEi-fY6rE&t=302s) about a kobra headless piston line maker:

> I'll try to explain how the sand duping works:
> First, bottom piston retracts, sand gets updated and schedules a tiletick for 2 gt and then, 2 gt later, it creates a sand entity in its position at tiletick phase. At block event phase, sand block gets pulled. Then the tricky part comes at entity phase: first (because it is older) TNT entity explodes and gives momentum to the sand entity, then when sand entity is ticked it first increments 'time' variable by one (life time), then it moves itself (because of the momentum), collides with the end portal and gets immediatly teleported, so a new sand entity is created at the End, all the data from the old sand entity is copied and this old one gets deleted (the important thing here is 'time' variable). If the end portal wasn't there, then the sand entity would delete itself the first time it gets ticked (when 'time' variable is equal to 1) because there isn't a sand block at its position anymore. Since sand entity's 'time' variable at the End was copied as 1, when it gets ticked it will be incremented by one again ('time' will be 2 now) so it won't check for the block in its position, therefore not deleting itself.
To make it clear, when sand entity is ticked it first increments 'time' variable by one, then it moves itself and then it checks if it is being ticked for the first time. If the code was different and it incremented 'time' variable after moving itself, then the sand entity at the End would disappear.

In spite of the comment being under the video it is under, it relates to [another video](https://youtu.be/4HEw2p5yMBg) by Mxi10 showcasing sand duping itself. The bug itself was, to our knowledge, first discovered by test137E29, who showed it in [this video](https://www.youtube.com/watch?v=7m2G1QeV-nQ).

I might write a more general and in-depth explanation later, but this can be a useful resource for now.
