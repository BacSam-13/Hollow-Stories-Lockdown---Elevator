The porpouse of this project, was to implement an asset intentend to be interactive with the player, while it displays animations, being able to be used in Level Sequences, and allow
the player to move among levels.

I am going to approach the process to implement this idea, with an already existing mesh for an elevator.

# Usage of already existing assets
First of all, it is neceesary to mention which elements were used, that were already made inside "Hollow Sense".
  - Actor Component: InteractableDoor
  - BP: InteractableObject

These components, were originally designed to allow the player to interact with doors and move to a designed level; and interact with an object, giving
different effects, depending of the object itself.

They were used for the next porpouses:
  - InteractableDoor: Give the player the option to interact inside the elevator, and move to a designed level.
  - InteractableObject: Allows the player to interact with the elevator from the outside, and opening it´s doors.

Now, it is neccesary to mention that, for the proper operation of the asset, it is required to use some animations, opening and closing
the doors of the elevator.
These animations are already implemented in the asset, so, for future explanations, we will refer to them such as: Anim_Open_Door and Anim_Close_Door.

These animations help to give the player a better inmersion on the elevator´s operation. And for such, we need to define where and when this animations
will be displayed, along to when and where, the already mentioned assets can be used by the player.

# Box Collisions
The main rules of the asset can be described below:
  - If the player is outside the elevator, the doors can be opened.
  - If the player is inside the elevator, they can move into a different level.
  - If the player leaves the elevator, the doors will close.
  - If the doors are moving, the player can´t interact with the outside of the elevator.
  - If the doors are opened, they can be opened again.

These ideas will be implemented using Box Collisions. This way, the assets know where the player is, in which context, and what they can do.
We created two Box Collisions:
  - Box_KeepOpen: Covers the inside of the elevator and near outside. It keeps the doors opened, if they were already opened.
  - Box_Interact_Botonera_Int: Inside the elevator, allows to interact with "InteractableDoor".

# Cases
Now that we have the box collisions to work with, we need to define what to do in different cases. This will give
the asset enough information to know when keep the doors opened, let the player interact with some componentns, etc...

But first, we need to define some variables that will be required during all the cases:
```text
CanInteract = true
KeepOpen = false
IsOpen = false
```

## Begin Play
### Explanation
Once we start the game, we allow the player to interact with InteractableObject, and by how it is defined, it can be only done from the outside.
We also, turn off and on the collisions from Box_Interact_Botonera_Int, allowing the game to detect early if ther player spawned inside the elevator, and open the doors for
them.
```text
SET InteractableObject.CanInteract = CanInteract
DISABLE Box_Interact_Botonera_Int.Collisions
DELAY UNTIL NEXT TICK
ENABLE Box_Interact_Botonera_Int.Collisions
```

## Doors Closing
### Explanation
This case is when the elevator is starting to get it´s doors closed. And once they are closed, allowing the player to interact with InteractanleObject.
```text
SET IsOpen = false
PLAY Anim_Close_Door
SET InteractableObject.CanInteract = true
```

## Doors Opening
### Explanation
This case is when the elevator is starting to get it´s doors opened. And once they are opened, check if they should stay opened.
```text
SET KeepOpen = true
SET IsOpen = true
PLAY Anim_Open_Door
IF KeepOpen:
    KeepOpen = false
ELSE:
    CALL CASE Doors Closing
END IF
END ELSE
```

## Box_KeepOpen Begin Overlap with Player
### Explanation
If the player is inside this area while the doors are opening, it keeps them opened.
```text
IF IsOpen:
    SET KeepOpen = true
ENF IF
```

## Box_KeepOpen End Overlap with Player
### Explanation
If the player leaves this area, and the doors doesn´t have to keep open, they close.
```text
IF IsOpen:
    IF KeepOpen:
        SET KeepOpen = false
    ELSE:
        CALL CASE Doors Closing
    END ELSE
    END IF
```

## Box_Interact_Botonera_Int Begin Overlap with Player
### Explanation
It allows the player to interact with the elevator, and move into a different level, and in case the player
spawned inside the elevator, it opens the doors for them.
```text
IF KeepOpen:
    SET InteractableDoor.CanInteract = true
ELSE:
    CALL CASE Doors Opening
END ELSE
END IF
```

## Box_Interact_Botonera_Int End Overlap with Player
### Explanation
If the players leaves the elevator, they can no longer move into a different level.
```text
SET InteractableDoor.CanInteract = false
```
