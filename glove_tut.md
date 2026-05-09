\# How To Add The Gardening Glove As A Separate Item #



This tutorial is divided into 4 parts:



1\. \*\*Adding the Glove button\*\*

2\. \*\*Making the Glove actually work\*\*

3\. \*\*Making it unlockable through Crazy Dave's Shop\*\*

4\. \*\*Making it unlockable after a certain Adventure level\*\*



\# Part 1: Adding the Glove button



This will teach us how to add the Glove.



\## Step 1: Add the function declarations



In `Board.h` near `DrawShovel` and `GetShovelButtonRect` add:



```cpp

void DrawGlove(Graphics\* g);

Rect GetGloveButtonRect();

```



These are the functions we need.



\## Step 2: Add the helper function and button rect



In `Board.cpp` near `GetShovelButtonRect()` add:



```cpp

static bool BoardShouldShowSeparateGlove(Board\* theBoard)

{

&#x20;   return theBoard->mShowShovel \&\&

&#x20;       theBoard->mApp->mGameMode != GameMode::GAMEMODE\_CHALLENGE\_ZEN\_GARDEN \&\&

&#x20;       theBoard->mApp->mGameMode != GameMode::GAMEMODE\_TREE\_OF\_WISDOM;

}



Rect Board::GetGloveButtonRect()

{

&#x20;   Rect aRect = GetShovelButtonRect();

&#x20;   aRect.mX += Sexy::IMAGE\_SHOVELBANK->GetWidth() - 8;

&#x20;   return aRect;

}

```



\- `BoardShouldShowSeparateGlove(...)` decides when the glove appears.

\- `GetGloveButtonRect()` decides where the glove goes.



\## Step 3: Draw the glove



In `Board.cpp` near `DrawShovel(Graphics\* g)` add:



```cpp

void Board::DrawGlove(Graphics\* g)

{

&#x20;   Rect aGloveRect = GetGloveButtonRect();

&#x20;   g->DrawImage(Sexy::IMAGE\_SHOVELBANK, aGloveRect.mX, aGloveRect.mY);



&#x20;   if (mCursorObject->mCursorType != CursorType::CURSOR\_TYPE\_GLOVE \&\&

&#x20;       mCursorObject->mCursorType != CursorType::CURSOR\_TYPE\_PLANT\_FROM\_GLOVE \&\&

&#x20;       mCursorObject->mCursorType != CursorType::CURSOR\_TYPE\_PLANT\_FROM\_WHEEL\_BARROW)

&#x20;   {

&#x20;       g->DrawImage(Sexy::IMAGE\_ZEN\_GARDENGLOVE, aGloveRect.mX - 6, aGloveRect.mY - 4);

&#x20;   }

}

```



\## Step 4: Use the draw glove function



In the board drawing section of `Board.cpp`, directly after:



```cpp

DrawShovel(g);

```



add:



```cpp

if (BoardShouldShowSeparateGlove(this))

{

&#x20;   DrawGlove(g);

}

```



\## Step 5: Make the glove clickable



In `Board::MouseHitTest(...)` inside `Board.cpp` add this directly after the shovel hit test:



```cpp

Rect aGloveButtonRect = GetGloveButtonRect();

if (BoardShouldShowSeparateGlove(this) \&\& aGloveButtonRect.Contains(x, y) \&\& CanInteractWithBoardButtons())

{

&#x20;   theHitResult->mObjectType = GameObjectType::OBJECT\_TYPE\_GLOVE;

&#x20;   return true;

}

```



\## Step 6: Add a tooltip



In the tooltip section of `Board.cpp` add:



```cpp

if (aHitResult.mObjectType == GameObjectType::OBJECT\_TYPE\_GLOVE \&\& BoardShouldShowSeparateGlove(this))

{

&#x20;   mToolTip->SetLabel(\_S("\[GLOVE\_TOOLTIP]"));

&#x20;   Rect aGloveButtonRect = GetGloveButtonRect();

&#x20;   mToolTip->mX = aGloveButtonRect.mX + 35;

&#x20;   mToolTip->mY = aGloveButtonRect.mY + 72;

&#x20;   mToolTip->mCenter = true;

&#x20;   mToolTip->mVisible = true;

&#x20;   return;

}

```



\## Step 7: Prevent duplicate glove rendering



In `DrawZenButtons(...)` inside `Board.cpp`, find the glove case and replace it with:



```cpp

else if (aTool == GameObjectType::OBJECT\_TYPE\_GLOVE)

{

&#x20;   if (BoardShouldShowSeparateGlove(this))

&#x20;   {

&#x20;       continue;

&#x20;   }



&#x20;   if (mCursorObject->mCursorType != CursorType::CURSOR\_TYPE\_PLANT\_FROM\_GLOVE \&\&

&#x20;       mCursorObject->mCursorType != CursorType::CURSOR\_TYPE\_PLANT\_FROM\_WHEEL\_BARROW)

&#x20;   {

&#x20;       g->DrawImage(Sexy::IMAGE\_ZEN\_GARDENGLOVE, aButtonRect.mX - 6, aButtonRect.mY - 4);

&#x20;   }

}

```



This stops the glove from being drawn twice.



\# Part 2: Making the Glove actually work



This will make the Glove work.



\## Step 1: Make the glove pick a plant up



In `Board.cpp`, find the function `Board::MouseDownWithTool(...)`.



Inside the part that already checks for `theCursorType == CursorType::CURSOR\_TYPE\_SHOVEL`, add this glove branch right after the shovel branch:



```cpp

else if (theCursorType == CursorType::CURSOR\_TYPE\_GLOVE)

{

&#x20;   mCursorObject->mType = aPlant->mSeedType;

&#x20;   mCursorObject->mImitaterType = aPlant->mImitaterType;

&#x20;   mCursorObject->mGlovePlantID = (PlantID)mPlants.DataArrayGetID(aPlant);

&#x20;   mCursorObject->mCursorType = CursorType::CURSOR\_TYPE\_PLANT\_FROM\_GLOVE;

&#x20;   mApp->PlaySample(SOUND\_TAP);

&#x20;   return;

}

```



This stores the plant data.



`return;` stops it from falling through to `ClearCursor()`.



`mApp->PlaySample(SOUND\_TAP);` is the glove sound effect.



\## Step 2: Make the carried plant show on the cursor



In `CursorObject.cpp`, there are \*\*two\*\* places you need to update.



\### First place



Find the `case CursorType::CURSOR\_TYPE\_PLANT\_FROM\_GLOVE:` block inside `CursorObject::Draw(...)`.



Replace it with this:



```cpp

case CursorType::CURSOR\_TYPE\_PLANT\_FROM\_GLOVE:

{

&#x20;   float aOffsetX = -10.0f;

&#x20;   float aOffsetY = PlantDrawHeightOffset(mBoard, nullptr, mType, -1, -1) - 10.0f;

&#x20;   if (Plant::IsFlying(mType) || mType == SeedType::SEED\_GRAVEBUSTER)

&#x20;   {

&#x20;       aOffsetY += 30.0f;

&#x20;   }

&#x20;   aOffsetY -= 15.0f;



&#x20;   Plant::DrawSeedType(g, mType, mImitaterType, DrawVariation::VARIATION\_NORMAL, aOffsetX, aOffsetY);

&#x20;   break;

}

```



\### Second place



Find the `else if (mBoard->mCursorObject->mCursorType == CursorType::CURSOR\_TYPE\_PLANT\_FROM\_GLOVE)` block inside `CursorPreview::Draw(...)`.



Replace it with this:



```cpp

else if (mBoard->mCursorObject->mCursorType == CursorType::CURSOR\_TYPE\_PLANT\_FROM\_GLOVE)

{

&#x20;   if (mBoard->mApp->mGameMode == GameMode::GAMEMODE\_CHALLENGE\_ZEN\_GARDEN)

&#x20;   {

&#x20;       Plant\* aPlant = mBoard->mPlants.DataArrayTryToGet((unsigned int)mBoard->mCursorObject->mGlovePlantID);

&#x20;       if (aPlant \&\& aPlant->mPottedPlantIndex >= 0)

&#x20;       {

&#x20;           aPottedPlant = \&mApp->mPlayerInfo->mPottedPlant\[aPlant->mPottedPlantIndex];

&#x20;       }

&#x20;   }

}

```



The first change makes the main cursor draw the grabbed plant.



The second change fixes the placement preview logic.



So this step does two things:



\- makes the glove-held plant look correct on the main cursor

\- prevents the preview code from reading invalid potted-plant data in regular levels



\## Step 3: Make the glove place the plant back down



In `Board.cpp`, find the plant placement section where the game checks:



```cpp

if (mCursorObject->mCursorType == CursorType::CURSOR\_TYPE\_PLANT\_FROM\_GLOVE)

```



Replace the old Zen Garden move call with this:



```cpp

if (mCursorObject->mCursorType == CursorType::CURSOR\_TYPE\_PLANT\_FROM\_GLOVE)

{

&#x20;   Plant\* aPlant = mPlants.DataArrayTryToGet(mCursorObject->mGlovePlantID);

&#x20;   if (aPlant)

&#x20;   {

&#x20;       int aOldGridX = aPlant->mPlantCol;

&#x20;       int aOldGridY = aPlant->mRow;

&#x20;       int aPosX = GridToPixelX(aGridX, aGridY);

&#x20;       int aPosY = GridToPixelY(aGridX, aGridY);

&#x20;       float aDeltaX = aPosX - aPlant->mX;

&#x20;       float aDeltaY = aPosY - aPlant->mY;

&#x20;       bool aWasAsleep = aPlant->mIsAsleep;



&#x20;       aPlant->SetSleeping(false);



&#x20;       Plant\* aUnderPlant = GetTopPlantAt(aOldGridX, aOldGridY, PlantPriority::TOPPLANT\_ONLY\_UNDER\_PLANT);

&#x20;       if (aUnderPlant)

&#x20;       {

&#x20;           aUnderPlant->mX = aPosX;

&#x20;           aUnderPlant->mY = aPosY;

&#x20;           aUnderPlant->mPlantCol = aGridX;

&#x20;           aUnderPlant->mRow = aGridY;

&#x20;           aUnderPlant->mRenderOrder = aUnderPlant->CalcRenderOrder();

&#x20;           aUnderPlant->UpdateReanim();

&#x20;       }



&#x20;       aPlant->mX = aPosX;

&#x20;       aPlant->mY = aPosY;

&#x20;       aPlant->mPlantCol = aGridX;

&#x20;       aPlant->mRow = aGridY;

&#x20;       aPlant->mRenderOrder = aPlant->CalcRenderOrder();

&#x20;       aPlant->UpdateReanim();



&#x20;       TodParticleSystem\* aParticle = mApp->ParticleTryToGet(aPlant->mParticleID);

&#x20;       if (aParticle \&\& aParticle->mEmitterList.mSize)

&#x20;       {

&#x20;           TodParticleEmitter\* aEmitter = aParticle->mParticleHolder->mEmitters.DataArrayGet((unsigned int)aParticle->mEmitterList.GetHead()->mValue);

&#x20;           aParticle->SystemMove(aEmitter->mSystemCenter.x + aDeltaX, aEmitter->mSystemCenter.y + aDeltaY);

&#x20;       }



&#x20;       if (aWasAsleep)

&#x20;       {

&#x20;           aPlant->SetSleeping(true);

&#x20;       }



&#x20;       mApp->PlayFoley(FoleyType::FOLEY\_DROP);

&#x20;       DoPlantingEffects(aGridX, aGridY, aUnderPlant ? aUnderPlant : aPlant);

&#x20;   }

}

```



This moves the grabbed plant to the new tile.



Instead of calling Zen Garden movement code, it updates:



\- the plant's old and new grid position

\- the plant's X position

\- the plant's Y position

\- the plant's grid column

\- the plant's row

\- the plant's render order

\- the plant's reanimation state

\- the plant's particle position, if it has one

\- the under-plant too, if the moved plant is sitting on one

\- the drop sound effect



The important part here is using:



\- `CalcRenderOrder()`

\- `UpdateReanim()`



instead of only changing the coordinates.



This avoids crashes from half-updated plant visuals or state.



`mApp->PlayFoley(FoleyType::FOLEY\_DROP);` is the drop sound effect.



\## Step 4: Keep the plant-in-cursor behavior enabled



In `Board.cpp`, find the helper that checks whether the cursor is carrying a plant.



In this codebase, that helper looks like this:



```cpp

bool Board::IsPlantInCursor()

{

&#x20;   return 

&#x20;       mCursorObject->mCursorType == CursorType::CURSOR\_TYPE\_PLANT\_FROM\_BANK || 

&#x20;       mCursorObject->mCursorType == CursorType::CURSOR\_TYPE\_PLANT\_FROM\_USABLE\_COIN || 

&#x20;       mCursorObject->mCursorType == CursorType::CURSOR\_TYPE\_PLANT\_FROM\_GLOVE || 

&#x20;       mCursorObject->mCursorType == CursorType::CURSOR\_TYPE\_PLANT\_FROM\_DUPLICATOR || 

&#x20;       mCursorObject->mCursorType == CursorType::CURSOR\_TYPE\_PLANT\_FROM\_WHEEL\_BARROW;

}

```



Make sure this line exists inside it:



```cpp

mCursorObject->mCursorType == CursorType::CURSOR\_TYPE\_PLANT\_FROM\_GLOVE ||

```



This makes the board treat the glove-held plant like a real carried plant.



The board also checks whether the cursor is carrying a plant.



If `CURSOR\_TYPE\_PLANT\_FROM\_GLOVE` is missing from this helper, then the game may still do all of these things:



\- show the glove button

\- let the glove pick up a plant

\- change the cursor visually



but still fail to place the plant back down correctly, because the board never recognizes the glove-held plant as part of the normal plant-in-cursor flow.



This connects the glove to the normal plant placement system.



\# Part 3: Making it unlockable through Crazy Dave's Shop (optional)



This will make the Glove unlock through Crazy Dave's Shop.



\## Step 1: Add the purchase check



Replace the helper function from Part 1 with this version:



```cpp

static bool BoardShouldShowSeparateGlove(Board\* theBoard)

{

&#x20;   return theBoard->mShowShovel \&\&

&#x20;       theBoard->mApp->mGameMode != GameMode::GAMEMODE\_CHALLENGE\_ZEN\_GARDEN \&\&

&#x20;       theBoard->mApp->mGameMode != GameMode::GAMEMODE\_TREE\_OF\_WISDOM \&\&

&#x20;       theBoard->mApp->mPlayerInfo->mPurchases\[(int)StoreItem::STORE\_ITEM\_GARDENING\_GLOVE] > 0;

}

```



Now the glove only appears if the player purchased it.



That means:



\- if the player has not bought it, the glove button does not appear

\- if the player has bought it, the glove button appears beside the shovel



\## Step 2: Make sure the glove is tied to the correct store item



The glove purchase should already exist in the shop code as:



```cpp

STORE\_ITEM\_GARDENING\_GLOVE

```



You do not need to invent a new item if your project already has this one. You are just reusing the purchase flag that Crazy Dave's Shop already saves.



This keeps it simple.



\## Step 3: Update `CanUseGameObject(...)`



In `Board::CanUseGameObject(...)`, make sure this exists:



```cpp

if (theGameObject == GameObjectType::OBJECT\_TYPE\_GLOVE)

{

&#x20;   return mApp->mPlayerInfo->mPurchases\[(int)StoreItem::STORE\_ITEM\_GARDENING\_GLOVE] > 0;

}

```



This keeps the board's usability checks consistent with the unlock condition.



It also keeps the general "can this object be used?" logic consistent.



\## Step 4: Leave the rest of the glove code alone



Do not rewrite the drawing code, hit test code, tooltip code, or cursor code from Parts 1 and 2.



Those parts should stay the same:



\- `DrawGlove(...)` still draws the button

\- `GetGloveButtonRect()` still places the button

\- `MouseHitTest(...)` still detects clicks on it

\- the tooltip code still shows the tooltip

\- the glove still uses the existing cursor and placement logic



The only real difference here is the purchase check.



This keeps the tutorial modular.



That is the best design because:



\- Part 1 teaches how to add the glove button

\- Part 2 teaches how to make it work

\- Part 3 teaches one possible unlock method



\# Part 4: Making it unlockable after a certain Adventure level (optional)



Don't want to use Crazy Dave's Shop? We can unlock the Glove after a level instead.



Keep Parts 1 and 2 the same and only change `BoardShouldShowSeparateGlove(...)`.



\## Step 1: Replace the helper function with an Adventure progress check



Replace the helper function in `Board.cpp` with something like this:



```cpp

static bool BoardShouldShowSeparateGlove(Board\* theBoard)

{

&#x20;   return theBoard->mShowShovel \&\&

&#x20;       theBoard->mApp->mGameMode != GameMode::GAMEMODE\_CHALLENGE\_ZEN\_GARDEN \&\&

&#x20;       theBoard->mApp->mGameMode != GameMode::GAMEMODE\_TREE\_OF\_WISDOM \&\&

&#x20;       theBoard->mApp->mPlayerInfo->mLevel >= 25;

}

```



This unlocks the Glove at level `25`.



You can change `25` to whatever Adventure level you want.



For example:



\- `mLevel >= 10` unlocks it earlier

\- `mLevel >= 30` unlocks it later



You can still use Crazy Dave text or dialog to tell the player about the unlock.



\## Step 2: Keep the rest of the glove system the same



Just like the shop version, do not rewrite the actual glove feature.



Keep all of Parts 1 and 2 the same:



\- the glove rect

\- the glove draw function

\- the glove hit test

\- the glove pickup logic

\- the glove cursor logic

\- the glove placement logic



Only the unlock condition changes.



This keeps the system reusable.



That means the glove can be unlocked by:



\- Adventure progress

\- Crazy Dave's Shop

\- a menu option

\- a custom save flag

\- anything else



without changing how the glove itself works.



\## Optional Step 3: Use a custom save flag instead of raw level progress



If you want a more expandable setup, you could save a dedicated flag when the player reaches a certain level, and then check that flag instead.



For example, the helper might look like this:



```cpp

static bool BoardShouldShowSeparateGlove(Board\* theBoard)

{

&#x20;   return theBoard->mShowShovel \&\&

&#x20;       theBoard->mApp->mGameMode != GameMode::GAMEMODE\_CHALLENGE\_ZEN\_GARDEN \&\&

&#x20;       theBoard->mApp->mGameMode != GameMode::GAMEMODE\_TREE\_OF\_WISDOM \&\&

&#x20;       theBoard->mApp->mPlayerInfo->mHasUnlockedGlove;

}

```



This is cleaner if you want the Glove to stay unlocked.



