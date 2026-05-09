# How To Add The Gardening Glove As A Separate Item

This tutorial is divided into 3 parts:

1. **Part 1** explains how to add the Gardening Glove as its own in-level HUD item beside the shovel.
2. **Part 2** explains how to make the glove unlockable through Crazy Dave's Shop.
3. **Part 3** provides a short conclusion and design notes.



# Part 1: Creating The Separate Glove Button

This section assumes the Gardening Glove should always be available during normal gameplay. Unlock conditions will be covered later.



## Step 1: Add the function declarations

In `Lawn/Board.h`, near `DrawShovel` and `GetShovelButtonRect`, add:

```cpp
void DrawGlove(Graphics* g);
Rect GetGloveButtonRect();
```

### What this does

This adds support for a second tool button alongside the shovel.



## Step 2: Add the helper function and button rect

In `Lawn/Board.cpp`, near `GetShovelButtonRect()`, add:

```cpp
static bool BoardShouldShowSeparateGlove(Board* theBoard)
{
    return theBoard->mShowShovel &&
        theBoard->mApp->mGameMode != GameMode::GAMEMODE_CHALLENGE_ZEN_GARDEN &&
        theBoard->mApp->mGameMode != GameMode::GAMEMODE_TREE_OF_WISDOM;
}

Rect Board::GetGloveButtonRect()
{
    Rect aRect = GetShovelButtonRect();
    aRect.mX += Sexy::IMAGE_SHOVELBANK->GetWidth() - 8;
    return aRect;
}
```

### What this does

- `BoardShouldShowSeparateGlove(...)` determines whether the glove button should be visible.
- `GetGloveButtonRect()` positions the glove beside the shovel button.

 

## Step 3: Draw the glove

In `Lawn/Board.cpp`, near `DrawShovel(Graphics* g)`, add:

```cpp
void Board::DrawGlove(Graphics* g)
{
    Rect aGloveRect = GetGloveButtonRect();
    g->DrawImage(Sexy::IMAGE_SHOVELBANK, aGloveRect.mX, aGloveRect.mY);

    if (mCursorObject->mCursorType != CursorType::CURSOR_TYPE_GLOVE &&
        mCursorObject->mCursorType != CursorType::CURSOR_TYPE_PLANT_FROM_GLOVE &&
        mCursorObject->mCursorType != CursorType::CURSOR_TYPE_PLANT_FROM_WHEEL_BARROW)
    {
        g->DrawImage(Sexy::IMAGE_ZEN_GARDENGLOVE, aGloveRect.mX - 6, aGloveRect.mY - 4);
    }
}
```

### What this does

This draws the Gardening Glove as a separate HUD tool using the same button background as the shovel.

 

## Step 4: Draw the glove in the board UI

In the board drawing section of `Lawn/Board.cpp`, directly after:

```cpp
DrawShovel(g);
```

add:

```cpp
if (BoardShouldShowSeparateGlove(this))
{
    DrawGlove(g);
}
```

### What this does

This makes the glove appear during gameplay.

 

## Step 5: Make the glove clickable

In `Board::MouseHitTest(...)` inside `Lawn/Board.cpp`, add this directly after the shovel hit test:

```cpp
Rect aGloveButtonRect = GetGloveButtonRect();
if (BoardShouldShowSeparateGlove(this) && aGloveButtonRect.Contains(x, y) && CanInteractWithBoardButtons())
{
    theHitResult->mObjectType = GameObjectType::OBJECT_TYPE_GLOVE;
    return true;
}
```

### What this does

This allows the board to recognize clicks on the glove button.

 

## Step 6: Add a tooltip

In the tooltip section of `Lawn/Board.cpp`, add:

```cpp
if (aHitResult.mObjectType == GameObjectType::OBJECT_TYPE_GLOVE && BoardShouldShowSeparateGlove(this))
{
    mToolTip->SetLabel(_S("[GLOVE_TOOLTIP]"));
    Rect aGloveButtonRect = GetGloveButtonRect();
    mToolTip->mX = aGloveButtonRect.mX + 35;
    mToolTip->mY = aGloveButtonRect.mY + 72;
    mToolTip->mCenter = true;
    mToolTip->mVisible = true;
    return;
}
```

### What this does

This adds a tooltip for the separate glove button.

 

## Step 7: Prevent duplicate glove rendering

In `DrawZenButtons(...)` inside `Lawn/Board.cpp`, find the glove case and replace it with:

```cpp
else if (aTool == GameObjectType::OBJECT_TYPE_GLOVE)
{
    if (BoardShouldShowSeparateGlove(this))
    {
        continue;
    }

    if (mCursorObject->mCursorType != CursorType::CURSOR_TYPE_PLANT_FROM_GLOVE &&
        mCursorObject->mCursorType != CursorType::CURSOR_TYPE_PLANT_FROM_WHEEL_BARROW)
    {
        g->DrawImage(Sexy::IMAGE_ZEN_GARDENGLOVE, aButtonRect.mX - 6, aButtonRect.mY - 4);
    }
}
```

### What this does

Without this change, the glove would render twice:
- once in the normal board HUD
- once in the Zen Garden tool loop

This prevents duplicate rendering.

 

# Result of Part 1

At this point, the Gardening Glove:

- has its own HUD button
- appears beside the shovel
- is clickable
- has its own tooltip
- reuses the existing glove cursor logic
- no longer replaces the shovel

This creates a clean base implementation that can later support unlock systems or custom conditions.

 

# Part 2: Making The Glove Unlockable

Now that the glove works as a standalone feature, we can make it unlockable through Crazy Dave’s Shop.

 

## Step 1: Add the purchase check

Replace the helper function from Part 1 with this version:

```cpp
static bool BoardShouldShowSeparateGlove(Board* theBoard)
{
    return theBoard->mShowShovel &&
        theBoard->mApp->mGameMode != GameMode::GAMEMODE_CHALLENGE_ZEN_GARDEN &&
        theBoard->mApp->mGameMode != GameMode::GAMEMODE_TREE_OF_WISDOM &&
        theBoard->mApp->mPlayerInfo->mPurchases[(int)StoreItem::STORE_ITEM_GARDENING_GLOVE] > 0;
}
```

### What this does

The glove now only appears if the player has purchased the Gardening Glove upgrade.

 

## Step 2: Reuse the existing store item

The game already includes the purchase entry:

```cpp
STORE_ITEM_GARDENING_GLOVE
```

Because of this, no new shop item needs to be created. The tutorial simply reuses the existing purchase flag.

 

## Step 3: Update `CanUseGameObject(...)`

In `Board::CanUseGameObject(...)`, make sure this exists:

```cpp
if (theGameObject == GameObjectType::OBJECT_TYPE_GLOVE)
{
    return mApp->mPlayerInfo->mPurchases[(int)StoreItem::STORE_ITEM_GARDENING_GLOVE] > 0;
}
```

### What this does

This keeps the board’s internal usability checks consistent with the unlock condition.

 

## Step 4: Leave the rest of the implementation unchanged

The remaining code from Part 1 should stay exactly the same:

- `DrawGlove(...)`
- `GetGloveButtonRect()`
- `MouseHitTest(...)`
- tooltip logic
- cursor handling

The only difference is the visibility condition.

This separation keeps the system modular and easier to reuse in other mods.

 

# Result of Part 2

The Gardening Glove now behaves exactly like before, except it only appears after purchase.

This design keeps the feature modular:

- **Part 1** creates the glove system
- **Part 2** adds one possible unlock method

If desired, the unlock condition can later be replaced with:
- level progression
- achievements
- challenge rewards
- configuration files
- custom save flags
- anything else

 

# Part 3: Conclusion

The cleanest way to implement features like the Gardening Glove is to separate the system itself from its unlock conditions.

First:
- create the feature
- draw it
- make it functional
- connect it to existing game logic

Then:
- decide how the player gains access to it

This approach keeps the code reusable, modular, and easier to maintain across different mods.

If a project does not need Crazy Dave’s Shop integration, Part 1 alone is enough.

If shop progression is desired, continue to Part 2.
