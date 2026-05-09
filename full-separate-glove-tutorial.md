# How To Add The Gardening Glove As A Separate Item #

This tutorial is divided into 3 parts:

1. **Adding the Glove** 
2. **Making it unlockable**
3. **Conclusion**

# Part 1: Creating The Separate Glove Button

This section of the tutorial will tell you how to add the Glove tool

## Step 1: Add the function declarations

In `Lawn/Board.h` near `DrawShovel` and `GetShovelButtonRect` add:

```cpp
void DrawGlove(Graphics* g);
Rect GetGloveButtonRect();
```

### What this does

This adds support for a second tool button alongside the shovel.



## Step 2: Add the helper function and button rect

In `Lawn/Board.cpp` near `GetShovelButtonRect()` add:

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

- `BoardShouldShowSeparateGlove(...)` This tells the game when the glove shall appear. 
- `GetGloveButtonRect()`  this tells the game where the glove should go. 

 

## Step 3: Draw the glove

In `Lawn/Board.cpp` near `DrawShovel(Graphics* g)` add:

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

It draws the glove. 
 

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

This makes the glove appear in gameplay.

 

## Step 5: Make the glove clickable

In `Board::MouseHitTest(...)` inside `Lawn/Board.cpp` add this directly after the shovel hit test:

```cpp
Rect aGloveButtonRect = GetGloveButtonRect();
if (BoardShouldShowSeparateGlove(this) && aGloveButtonRect.Contains(x, y) && CanInteractWithBoardButtons())
{
    theHitResult->mObjectType = GameObjectType::OBJECT_TYPE_GLOVE;
    return true;
}
```

### What this does

It makes it so that the glove can be clickable. 
 
## Step 6: Add a tooltip

In the tooltip section of `Lawn/Board.cpp` add:

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

This adds a tooltip for the glove.

## Step 7: Prevent duplicate glove rendering

In `DrawZenButtons(...)` inside `Lawn/Board.cpp` find the glove case and replace it with:

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

This is so that way it prevents itself from appearing twice. 

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

Now the glove will appear if the player purchased it. 

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
Normally, I would say this but simplified, but I can't really find the best way to simplify it so yeah
 
# DONE yeah thats it for this Part

# Part 3: Conclusion
I don't really have anything to say for this part. I just thought it'd be cool if I added a conclusion for now. 

