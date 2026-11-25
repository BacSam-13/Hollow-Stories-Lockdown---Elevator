# Interactive Elevator System

The purpose of this project was to implement an interactive elevator asset that displays animations, can be used within Level Sequences, and allows the player to travel between levels.

To achieve this, we used an existing elevator mesh and integrated it with previously developed interaction systems. The following sections explain the implementation process and how the different components work together.

# Usage of Existing Assets

First, let's identify the existing elements from *Hollow Sense* that were reused in this project:

- **Actor Component:** `InteractableDoor`
- **Blueprint:** `InteractableObject`

These elements were originally designed to allow the player to interact with doors, travel to a designated level, and interact with objects that trigger different effects depending on their functionality.

For this project, they were used for the following purposes:

- **InteractableDoor:** Gives the player the option to interact with the elevator's interior control panel and travel to a designated level.
- **InteractableObject:** Allows the player to interact with the elevator from the outside, triggering its doors to open. Once activated, it becomes unavailable until the doors finish closing.

## Door Animations

For the elevator to operate properly, we needed animations to control the opening and closing of its doors.

These animations were already implemented in the asset. For the following explanations, we will refer to them as:

- `Anim_Open_Door`
- `Anim_Close_Door`

These animations provide visual feedback for the elevator's operation. We therefore needed to define when they should play and under which circumstances the player should be allowed to interact with the elevator.

# Box Collisions

The elevator's behavior is governed by the following rules:

- If the player is outside the elevator, they can interact with it to open the doors.
- If the player is inside the elevator, they can use the control panel to travel to another level.
- If the player leaves the elevator, its doors should close.
- While the doors are moving, the player cannot interact with the elevator from the outside.
- If the player is within the designated area, the doors should remain open.

To implement these rules, we used Box Collision components. These allow the elevator to detect the player's position and determine which interactions should be available.

We created two Box Collisions:

- **Box_KeepOpen:** Covers the elevator's interior and a nearby area outside it. It keeps the doors open if the player is within this area and the doors are already open.
- **Box_Interact_Botonera_Int:** Covers the elevator's interior control panel area. It detects when the player enters or leaves the area where `InteractableDoor` can be used.

# Elevator Behavior

Now that we have defined the collision areas, we can explore how the elevator behaves in different situations.

Before examining each case, let's define the variables used throughout the system:

```text
CanInteract = true
KeepOpen = false
IsOpen = false
```

- `CanInteract`: Determines whether the player can interact with `InteractableObject`.
- `KeepOpen`: Indicates whether the elevator doors should remain open after the opening sequence. This value may be used by the overlap events while the doors are opening, and even be modified by them.
- `IsOpen`: Tracks whether the elevator's doors are currently considered open.

## Begin Play

### Explanation

When the game starts, we allow the player to interact with `InteractableObject`, which is intended to be used from outside the elevator.

We also temporarily disable the collision of `Box_Interact_Botonera_Int` and re-enable it on the next tick. This allows the system to detect whether the player spawned inside the elevator and handle that situation accordingly.

```text
SET InteractableObject.CanInteract = CanInteract

DISABLE Box_Interact_Botonera_Int.Collisions
DELAY UNTIL NEXT TICK
ENABLE Box_Interact_Botonera_Int.Collisions
```

## Doors Closing

### Explanation

This case handles the elevator's closing sequence.

Once the closing animation finishes, the system allows the player to interact with `InteractableObject` again, enabling them to open the elevator from outside.

```text
SET IsOpen = false
PLAY Anim_Close_Door

WHEN Anim_Close_Door FINISHES:
    SET InteractableObject.CanInteract = true
```

## Doors Opening

### Explanation

This case handles the elevator's opening sequence.

We update the elevator's state, play the opening animation, and check whether the doors should remain open or start closing again.

While the doors are moving, outside interaction remains disabled.

```text
SET InteractableObject.CanInteract = false
SET KeepOpen = true
SET IsOpen = true

PLAY Anim_Open_Door

WHEN Anim_Open_Door FINISHES:
    IF KeepOpen:
        SET KeepOpen = false
    ELSE:
        CALL CASE Doors Closing
    END IF
```

## Box_KeepOpen Begin Overlap with Player

### Explanation

If the player enters this area while the doors are open or opening, the system marks the elevator as needing to remain open.

```text
IF IsOpen:
    SET KeepOpen = true
END IF
```

## Box_KeepOpen End Overlap with Player

### Explanation

When the player leaves this area, the system checks whether the doors should remain open.

If `KeepOpen` is true, the flag is reset. Otherwise, the elevator starts its closing sequence.

```text
IF IsOpen:
    IF KeepOpen:
        SET KeepOpen = false
    ELSE:
        CALL CASE Doors Closing
    END IF
END IF
```

## Box_Interact_Botonera_Int Begin Overlap with Player

### Explanation

When the player enters the elevator's interior control panel area, the system allows them to use `InteractableDoor` and travel to another level.

If the player spawned inside the elevator, this event also handles opening the doors for them.

```text
IF KeepOpen:
    SET InteractableDoor.CanInteract = true
ELSE:
    CALL CASE Doors Opening
END IF
```

## Box_Interact_Botonera_Int End Overlap with Player

### Explanation

When the player leaves the elevator's interior control panel area, they can no longer use `InteractableDoor` to travel to another level.

```text
SET InteractableDoor.CanInteract = false
```
