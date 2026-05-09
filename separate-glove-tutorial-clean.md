# How To Add The Gardening Glove As A Separate Item

This tutorial is divided into 3 parts:

1. **Part 1** explains how to add the Gardening Glove as its own in-level HUD item beside the shovel.
2. **Part 2** explains how to make the glove unlockable through Crazy Dave's Shop.
3. **Part 3** provides a short conclusion and design notes.

---

# Part 1: Creating The Separate Glove Button

This section assumes the Gardening Glove should always be available during normal gameplay. Unlock conditions will be covered later.

---

## Step 1: Add the function declarations

In `Lawn/Board.h`, near `DrawShovel` and `GetShovelButtonRect`, add:

```cpp
void DrawGlove(Graphics* g);
Rect GetGloveButtonRect();
```

### What this does

This adds support for a second tool button alongside the shovel.

---

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

---

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
