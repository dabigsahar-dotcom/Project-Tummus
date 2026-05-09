# How To Add The Gardening Glove As A Separate Item

This tutorial is split into 4 parts:

1. Part 1 explains how to make the glove itself as a separate in-level item beside the shovel.
2. Part 2 explains how to make that glove unlockable through Crazy Dave's Shop if you want.
3. Part 3 explains how to unlock the glove after a certain Adventure level instead, if you want.
4. Part 4 is a short conclusion.

---

## Part 1: How To Make The Glove

This part assumes the glove should always be available in normal levels. We are not making it unlockable yet.

### Step 1: Add the new function declarations

Put these declarations in `Lawn/Board.h`, near `DrawShovel` and `GetShovelButtonRect`:

    void DrawGlove(Graphics* g);
    Rect GetGloveButtonRect();

### What this does

This tells the board that it now has a second tool button besides the shovel.

---

### Step 2: Add a helper function and glove button rect

Put this in `Lawn/Board.cpp`, near `GetShovelButtonRect()`:

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

### What this does

- `BoardShouldShowSeparateGlove` decides when the glove should appear.
- `GetGloveButtonRect` places the glove right beside the shovel.

---

### Step 3: Add the glove draw function

Put this in `Lawn/Board.cpp`, near `DrawShovel(Graphics* g)`:

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

### What this does

This draws the glove as its own separate HUD item using the same bank art as the shovel.

---

### Step 4: Draw the glove in the board UI

In the board draw section of `Lawn/Board.cpp`, right after `DrawShovel(g);`, add:

    if (BoardShouldShowSeparateGlove(this))
    {
    	DrawGlove(g);
    }

### What this does

This makes the glove actually appear in-game.

---

### Step 5: Make the glove clickable

In `Board::MouseHitTest(...)` in `Lawn/Board.cpp`, right after the shovel button hit test, add:

    Rect aGloveButtonRect = GetGloveButtonRect();
    if (BoardShouldShowSeparateGlove(this) && aGloveButtonRect.Contains(x, y) && CanInteractWithBoardButtons())
    {
    	theHitResult->mObjectType = GameObjectType::OBJECT_TYPE_GLOVE;
    	return true;
    }

### What this does

This makes the glove button clickable so the board recognizes it as a glove item.

---

### Step 6: Add a tooltip for the glove

In the tooltip section of `Lawn/Board.cpp`, add:

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

### What this does

This gives the glove a proper tooltip in the new slot.

---

### Step 7: Prevent the glove from drawing twice

In `DrawZenButtons(...)` in `Lawn/Board.cpp`, find the glove case and change it to:

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

### What this does

This stops the glove from being drawn once in the board HUD and again in the Zen Garden tool loop.

---

### Result of Part 1

At this point, the glove is:

- its own separate item
- drawn beside the shovel
- clickable
- using its own tooltip
- not replacing the shovel

This is the best base setup because it works even if you want to unlock the glove some other way later.

---

## Part 2: How To Make The Glove Unlockable Through Crazy Dave's Shop

Now that the glove already works, we can make it unlockable through the shop.

### Step 1: Change the helper function so it checks for the purchase

Go back to the helper function from Part 1 and replace it with this version in `Lawn/Board.cpp`:

    static bool BoardShouldShowSeparateGlove(Board* theBoard)
    {
    	return theBoard->mShowShovel &&
    		theBoard->mApp->mGameMode != GameMode::GAMEMODE_CHALLENGE_ZEN_GARDEN &&
    		theBoard->mApp->mGameMode != GameMode::GAMEMODE_TREE_OF_WISDOM &&
    		theBoard->mApp->mPlayerInfo->mPurchases[(int)StoreItem::STORE_ITEM_GARDENING_GLOVE] > 0;
    }

### What this does

This changes the glove from "always show in normal levels" to "only show if the player owns the Gardening Glove purchase."

That means:

- if the player has not bought it, the glove button does not appear
- if the player has bought it, the glove button appears beside the shovel

---

### Step 2: Make sure the glove is tied to the correct store item

The glove purchase should already exist in the shop code as:

    STORE_ITEM_GARDENING_GLOVE

You do not need to invent a new item if your project already has this one. You are just reusing the purchase flag that Crazy Dave's Shop already saves.

### What this does

This keeps the tutorial simple. Instead of creating a whole new shop system, you only connect the glove visibility to a purchase that already exists.

---

### Step 3: Make sure the board still recognizes the glove as a usable game object

In `Board::CanUseGameObject(...)` in `Lawn/Board.cpp`, make sure this exists:

    if (theGameObject == GameObjectType::OBJECT_TYPE_GLOVE)
    {
    	return mApp->mPlayerInfo->mPurchases[(int)StoreItem::STORE_ITEM_GARDENING_GLOVE] > 0;
    }

### What this does

This keeps the rest of the game consistent with the unlock condition.

Even if your separate glove button is already checking the purchase, this makes sure the board's general "can this object be used?" logic also agrees with it.

---

### Step 4: Leave the rest of the glove code alone

Do not rewrite the drawing code, hit test code, tooltip code, or cursor code from Part 1.

Those parts should stay the same:

- `DrawGlove(...)` still draws the button
- `GetGloveButtonRect()` still places the button
- `MouseHitTest(...)` still detects clicks on it
- the tooltip code still shows the tooltip
- the glove still uses the existing cursor/tool logic

The only real difference in Part 2 is that now the helper function checks whether the player bought the glove.

### What this does

This keeps the tutorial modular.

That is the best design because:

- Part 1 teaches how to make the glove feature
- Part 2 teaches how to lock or unlock that feature

---

## Part 3: How To Unlock The Glove After A Certain Adventure Level

If you do not want to use Crazy Dave's Shop, you can unlock the glove after the player reaches or beats a certain Adventure level instead.

The cleanest way to do that is to keep all of Part 1 the same and only change the condition inside `BoardShouldShowSeparateGlove(...)`.

### Step 1: Replace the helper function with an Adventure progress check

Replace the helper function in `Lawn/Board.cpp` with something like this:

    static bool BoardShouldShowSeparateGlove(Board* theBoard)
    {
    	return theBoard->mShowShovel &&
    		theBoard->mApp->mGameMode != GameMode::GAMEMODE_CHALLENGE_ZEN_GARDEN &&
    		theBoard->mApp->mGameMode != GameMode::GAMEMODE_TREE_OF_WISDOM &&
    		theBoard->mApp->mPlayerInfo->mLevel >= 25;
    }

### What this does

This version unlocks the glove once the player's Adventure progress reaches level `25`.

You can change `25` to whatever Adventure level you want.

For example:

- `mLevel >= 10` unlocks it earlier
- `mLevel >= 30` unlocks it later

---

### Step 2: Adjust the level number based on how your project tracks progress

Some projects track Adventure progression a little differently.

For example, your project might use:

- the current Adventure level
- the highest unlocked Adventure level
- a completed-level flag

If `mPlayerInfo->mLevel` is not the exact field you want, replace that part with whatever value your project uses to track progression.

The important part is the pattern:

    some_progress_value >= your_unlock_level

### What this does

This gives modders flexibility. The tutorial teaches the structure even if a different codebase stores Adventure progress in a slightly different variable.

---

### Step 3: Keep the rest of the glove system the same

Just like the shop version, do not rewrite the actual glove feature.

Keep all of Part 1 the same:

- the glove rect
- the glove draw function
- the glove hit test
- the glove tooltip
- the glove cursor logic

Only the unlock condition changes.

### What this does

This keeps the system reusable.

That means the glove can be unlocked by:

- Adventure progress
- Crazy Dave's Shop
- a menu option
- a custom save flag
- anything else

without changing how the glove itself works.

---

### Optional Example: Use a custom save flag instead of raw level progress

If you want a more expandable setup, you could save a dedicated flag when the player reaches a certain level, and then check that flag instead.

For example, the helper might look like this:

    static bool BoardShouldShowSeparateGlove(Board* theBoard)
    {
    	return theBoard->mShowShovel &&
    		theBoard->mApp->mGameMode != GameMode::GAMEMODE_CHALLENGE_ZEN_GARDEN &&
    		theBoard->mApp->mGameMode != GameMode::GAMEMODE_TREE_OF_WISDOM &&
    		theBoard->mApp->mPlayerInfo->mHasUnlockedGlove;
    }

### What this does

This is cleaner if you want the glove to stay unlocked permanently for its own reason, instead of always tying it directly to the current Adventure level number.

---

## Part 4: Conclusion

The cleanest way to add features like the Gardening Glove is in layers.

First, make the feature work by itself:

- give it its own button
- draw it separately
- make it clickable
- make it reuse the existing cursor logic

Then, after it already works, decide how it gets unlocked:

- always available
- Crazy Dave's Shop
- Adventure progress
- custom quest
- minigame reward
- option toggle
- anything else

That way, the tutorial is reusable for other mods too.

If somebody wants the glove but does not want to use Crazy Dave's Shop, they can stop at Part 1 or use Part 3 instead.

If they do want the shop unlock, they can use Part 2.
