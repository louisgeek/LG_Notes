> 源码基于 androidx.recyclerview:recyclerview:1.2.0



自带 ViewHolder

局部刷新

自带 item 动画效果



侧重点不同，为啥没有进标准版里？









RecyclerView$ViewHolder



RecyclerView$Adapter



RecyclerView$Recycler





## ItemDecoration

### DividerItemDecoration.java

- 系统自带

```java
/**
 * DividerItemDecoration is a {@link RecyclerView.ItemDecoration} that can be used as a divider
 * between items of a {@link LinearLayoutManager}. It supports both {@link #HORIZONTAL} and
 * {@link #VERTICAL} orientations.
 *
 * <pre>
 *     mDividerItemDecoration = new DividerItemDecoration(recyclerView.getContext(),
 *             mLayoutManager.getOrientation());
 *     recyclerView.addItemDecoration(mDividerItemDecoration);
 * </pre>
 */
public class DividerItemDecoration extends RecyclerView.ItemDecoration {
    public static final int HORIZONTAL = LinearLayout.HORIZONTAL;
    public static final int VERTICAL = LinearLayout.VERTICAL;

    private static final String TAG = "DividerItem";
    private static final int[] ATTRS = new int[]{ android.R.attr.listDivider };

    private Drawable mDivider;

    /**
     * Current orientation. Either {@link #HORIZONTAL} or {@link #VERTICAL}.
     */
    private int mOrientation;

    private final Rect mBounds = new Rect();

    /**
     * Creates a divider {@link RecyclerView.ItemDecoration} that can be used with a
     * {@link LinearLayoutManager}.
     *
     * @param context Current context, it will be used to access resources.
     * @param orientation Divider orientation. Should be {@link #HORIZONTAL} or {@link #VERTICAL}.
     */
    public DividerItemDecoration(Context context, int orientation) {
        //获取 listDivider 
        final TypedArray a = context.obtainStyledAttributes(ATTRS);
        mDivider = a.getDrawable(0);
        if (mDivider == null) {
            Log.w(TAG, "@android:attr/listDivider was not set in the theme used for this "
                    + "DividerItemDecoration. Please set that attribute all call setDrawable()");
        }
        a.recycle();
        setOrientation(orientation);
    }

    /**
     * Sets the orientation for this divider. This should be called if
     * {@link RecyclerView.LayoutManager} changes orientation.
     *
     * @param orientation {@link #HORIZONTAL} or {@link #VERTICAL}
     */
    public void setOrientation(int orientation) {
        if (orientation != HORIZONTAL && orientation != VERTICAL) {
            throw new IllegalArgumentException(
                    "Invalid orientation. It should be either HORIZONTAL or VERTICAL");
        }
        mOrientation = orientation;
    }

    /**
     * Sets the {@link Drawable} for this divider.
     *
     * @param drawable Drawable that should be used as a divider.
     */
    public void setDrawable(@NonNull Drawable drawable) {
        if (drawable == null) {
            throw new IllegalArgumentException("Drawable cannot be null.");
        }
        mDivider = drawable;
    }

    /**
     * @return the {@link Drawable} for this divider.
     */
    @Nullable
    public Drawable getDrawable() {
        return mDivider;
    }

    @Override
    public void onDraw(Canvas c, RecyclerView parent, RecyclerView.State state) {
        if (parent.getLayoutManager() == null || mDivider == null) {
            return;
        }
        if (mOrientation == VERTICAL) {
            drawVertical(c, parent);
        } else {
            drawHorizontal(c, parent);
        }
    }

    private void drawVertical(Canvas canvas, RecyclerView parent) {
        canvas.save();
        final int left;
        final int right;
        //noinspection AndroidLintNewApi - NewApi lint fails to handle overrides.
        if (parent.getClipToPadding()) {
            left = parent.getPaddingLeft();
            right = parent.getWidth() - parent.getPaddingRight();
            canvas.clipRect(left, parent.getPaddingTop(), right,
                    parent.getHeight() - parent.getPaddingBottom());
        } else {
            left = 0;
            right = parent.getWidth();
        }

        final int childCount = parent.getChildCount();
        for (int i = 0; i < childCount; i++) {
            final View child = parent.getChildAt(i);
            parent.getDecoratedBoundsWithMargins(child, mBounds);
            final int bottom = mBounds.bottom + Math.round(child.getTranslationY());
            final int top = bottom - mDivider.getIntrinsicHeight();
            mDivider.setBounds(left, top, right, bottom);
            mDivider.draw(canvas);
        }
        canvas.restore();
    }

    private void drawHorizontal(Canvas canvas, RecyclerView parent) {
        canvas.save();
        final int top;
        final int bottom;
        //noinspection AndroidLintNewApi - NewApi lint fails to handle overrides.
        if (parent.getClipToPadding()) {
            top = parent.getPaddingTop();
            bottom = parent.getHeight() - parent.getPaddingBottom();
            canvas.clipRect(parent.getPaddingLeft(), top,
                    parent.getWidth() - parent.getPaddingRight(), bottom);
        } else {
            top = 0;
            bottom = parent.getHeight();
        }

        final int childCount = parent.getChildCount();
        for (int i = 0; i < childCount; i++) {
            final View child = parent.getChildAt(i);
            parent.getLayoutManager().getDecoratedBoundsWithMargins(child, mBounds);
            final int right = mBounds.right + Math.round(child.getTranslationX());
            final int left = right - mDivider.getIntrinsicWidth();
            mDivider.setBounds(left, top, right, bottom);
            mDivider.draw(canvas);
        }
        canvas.restore();
    }

    @Override
    public void getItemOffsets(Rect outRect, View view, RecyclerView parent,
            RecyclerView.State state) {
        if (mDivider == null) {
            outRect.set(0, 0, 0, 0);
            return;
        }
        if (mOrientation == VERTICAL) {
            outRect.set(0, 0, 0, mDivider.getIntrinsicHeight());
        } else {
            outRect.set(0, 0, mDivider.getIntrinsicWidth(), 0);
        }
    }
}
```

RecyclerView#draw 调用 RecyclerView#onDraw 然后会调 ItemDecoration#onDraw 方法，最后调 ItemDecoration#onDrawOver 方法



## RecyclerView#onMeasure

```java
  @Override
    protected void onMeasure(int widthSpec, int heightSpec) {
        if (mLayout == null) {
            defaultOnMeasure(widthSpec, heightSpec);
            return;
        }
        if (mLayout.isAutoMeasureEnabled()) {
            final int widthMode = MeasureSpec.getMode(widthSpec);
            final int heightMode = MeasureSpec.getMode(heightSpec);

            /**
             * This specific call should be considered deprecated and replaced with
             * {@link #defaultOnMeasure(int, int)}. It can't actually be replaced as it could
             * break existing third party code but all documentation directs developers to not
             * override {@link LayoutManager#onMeasure(int, int)} when
             * {@link LayoutManager#isAutoMeasureEnabled()} returns true.
             */
            mLayout.onMeasure(mRecycler, mState, widthSpec, heightSpec);

            // Calculate and track whether we should skip measurement here because the MeasureSpec
            // modes in both dimensions are EXACTLY.
            mLastAutoMeasureSkippedDueToExact =
                    widthMode == MeasureSpec.EXACTLY && heightMode == MeasureSpec.EXACTLY;
            if (mLastAutoMeasureSkippedDueToExact || mAdapter == null) {
                return;
            }

            if (mState.mLayoutStep == State.STEP_START) {
                dispatchLayoutStep1();
            }
            // set dimensions in 2nd step. Pre-layout should happen with old dimensions for
            // consistency
            mLayout.setMeasureSpecs(widthSpec, heightSpec);
            mState.mIsMeasuring = true;
            dispatchLayoutStep2();

            // now we can get the width and height from the children.
            mLayout.setMeasuredDimensionFromChildren(widthSpec, heightSpec);

            // if RecyclerView has non-exact width and height and if there is at least one child
            // which also has non-exact width & height, we have to re-measure.
            if (mLayout.shouldMeasureTwice()) {
                mLayout.setMeasureSpecs(
                        MeasureSpec.makeMeasureSpec(getMeasuredWidth(), MeasureSpec.EXACTLY),
                        MeasureSpec.makeMeasureSpec(getMeasuredHeight(), MeasureSpec.EXACTLY));
                mState.mIsMeasuring = true;
                dispatchLayoutStep2();
                // now we can get the width and height from the children.
                mLayout.setMeasuredDimensionFromChildren(widthSpec, heightSpec);
            }

            mLastAutoMeasureNonExactMeasuredWidth = getMeasuredWidth();
            mLastAutoMeasureNonExactMeasuredHeight = getMeasuredHeight();
        } else {
            //mAutoMeasure is disable
            if (mHasFixedSize) {
                //
                mLayout.onMeasure(mRecycler, mState, widthSpec, heightSpec);
                return;
            }
            // custom onMeasure
            if (mAdapterUpdateDuringMeasure) {
                startInterceptRequestLayout();
                onEnterLayoutOrScroll();
                processAdapterUpdatesAndSetAnimationFlags();
                onExitLayoutOrScroll();

                if (mState.mRunPredictiveAnimations) {
                    mState.mInPreLayout = true;
                } else {
                    // consume remaining updates to provide a consistent state with the layout pass.
                    mAdapterHelper.consumeUpdatesInOnePass();
                    mState.mInPreLayout = false;
                }
                mAdapterUpdateDuringMeasure = false;
                stopInterceptRequestLayout(false);
            } else if (mState.mRunPredictiveAnimations) {
                // If mAdapterUpdateDuringMeasure is false and mRunPredictiveAnimations is true:
                // this means there is already an onMeasure() call performed to handle the pending
                // adapter change, two onMeasure() calls can happen if RV is a child of LinearLayout
                // with layout_width=MATCH_PARENT. RV cannot call LM.onMeasure() second time
                // because getViewForPosition() will crash when LM uses a child to measure.
                setMeasuredDimension(getMeasuredWidth(), getMeasuredHeight());
                return;
            }

            if (mAdapter != null) {
                mState.mItemCount = mAdapter.getItemCount();
            } else {
                mState.mItemCount = 0;
            }
            startInterceptRequestLayout();
            //LayoutManager
            mLayout.onMeasure(mRecycler, mState, widthSpec, heightSpec);
            stopInterceptRequestLayout(false);
            mState.mInPreLayout = false; // clear
        }
    }

```



## RecyclerView#dispatchLayoutStep1

```java
  /**
     * The first step of a layout where we;
     * - process adapter updates
     * - decide which animation should run
     * - save information about current views
     * - If necessary, run predictive layout and save its information
     */
    private void dispatchLayoutStep1() {
        mState.assertLayoutStep(State.STEP_START);
        fillRemainingScrollValues(mState);
        mState.mIsMeasuring = false;
        startInterceptRequestLayout();
        mViewInfoStore.clear();
        onEnterLayoutOrScroll();
        processAdapterUpdatesAndSetAnimationFlags();
        saveFocusInfo();
        mState.mTrackOldChangeHolders = mState.mRunSimpleAnimations && mItemsChanged;
        mItemsAddedOrRemoved = mItemsChanged = false;
        mState.mInPreLayout = mState.mRunPredictiveAnimations;
        mState.mItemCount = mAdapter.getItemCount();
        findMinMaxChildLayoutPositions(mMinMaxLayoutPositions);

        if (mState.mRunSimpleAnimations) {
            // Step 0: Find out where all non-removed items are, pre-layout
            int count = mChildHelper.getChildCount();
            for (int i = 0; i < count; ++i) {
                final ViewHolder holder = getChildViewHolderInt(mChildHelper.getChildAt(i));
                if (holder.shouldIgnore() || (holder.isInvalid() && !mAdapter.hasStableIds())) {
                    continue;
                }
                final ItemHolderInfo animationInfo = mItemAnimator
                        .recordPreLayoutInformation(mState, holder,
                                ItemAnimator.buildAdapterChangeFlagsForAnimations(holder),
                                holder.getUnmodifiedPayloads());
                mViewInfoStore.addToPreLayout(holder, animationInfo);
                if (mState.mTrackOldChangeHolders && holder.isUpdated() && !holder.isRemoved()
                        && !holder.shouldIgnore() && !holder.isInvalid()) {
                    long key = getChangedHolderKey(holder);
                    // This is NOT the only place where a ViewHolder is added to old change holders
                    // list. There is another case where:
                    //    * A VH is currently hidden but not deleted
                    //    * The hidden item is changed in the adapter
                    //    * Layout manager decides to layout the item in the pre-Layout pass (step1)
                    // When this case is detected, RV will un-hide that view and add to the old
                    // change holders list.
                    mViewInfoStore.addToOldChangeHolders(key, holder);
                }
            }
        }
        if (mState.mRunPredictiveAnimations) {
            // Step 1: run prelayout: This will use the old positions of items. The layout manager
            // is expected to layout everything, even removed items (though not to add removed
            // items back to the container). This gives the pre-layout position of APPEARING views
            // which come into existence as part of the real layout.

            // Save old positions so that LayoutManager can run its mapping logic.
            saveOldPositions();
            final boolean didStructureChange = mState.mStructureChanged;
            mState.mStructureChanged = false;
            // temporarily disable flag because we are asking for previous layout
            mLayout.onLayoutChildren(mRecycler, mState);
            mState.mStructureChanged = didStructureChange;

            for (int i = 0; i < mChildHelper.getChildCount(); ++i) {
                final View child = mChildHelper.getChildAt(i);
                final ViewHolder viewHolder = getChildViewHolderInt(child);
                if (viewHolder.shouldIgnore()) {
                    continue;
                }
                if (!mViewInfoStore.isInPreLayout(viewHolder)) {
                    int flags = ItemAnimator.buildAdapterChangeFlagsForAnimations(viewHolder);
                    boolean wasHidden = viewHolder
                            .hasAnyOfTheFlags(ViewHolder.FLAG_BOUNCED_FROM_HIDDEN_LIST);
                    if (!wasHidden) {
                        flags |= ItemAnimator.FLAG_APPEARED_IN_PRE_LAYOUT;
                    }
                    final ItemHolderInfo animationInfo = mItemAnimator.recordPreLayoutInformation(
                            mState, viewHolder, flags, viewHolder.getUnmodifiedPayloads());
                    if (wasHidden) {
                        recordAnimationInfoIfBouncedHiddenView(viewHolder, animationInfo);
                    } else {
                        mViewInfoStore.addToAppearedInPreLayoutHolders(viewHolder, animationInfo);
                    }
                }
            }
            // we don't process disappearing list because they may re-appear in post layout pass.
            clearOldPositions();
        } else {
            clearOldPositions();
        }
        onExitLayoutOrScroll();
        stopInterceptRequestLayout(false);
        mState.mLayoutStep = State.STEP_LAYOUT;
    }

```





## RecyclerView#dispatchLayoutStep2

```java
 /**
     * The second layout step where we do the actual layout of the views for the final state.
     * This step might be run multiple times if necessary (e.g. measure).
     */
    private void dispatchLayoutStep2() {
        startInterceptRequestLayout();
        onEnterLayoutOrScroll();
        mState.assertLayoutStep(State.STEP_LAYOUT | State.STEP_ANIMATIONS);
        mAdapterHelper.consumeUpdatesInOnePass();
        mState.mItemCount = mAdapter.getItemCount();
        mState.mDeletedInvisibleItemCountSincePreviousLayout = 0;
        if (mPendingSavedState != null && mAdapter.canRestoreState()) {
            if (mPendingSavedState.mLayoutState != null) {
                mLayout.onRestoreInstanceState(mPendingSavedState.mLayoutState);
            }
            mPendingSavedState = null;
        }
        // Step 2: Run layout
        mState.mInPreLayout = false;
        mLayout.onLayoutChildren(mRecycler, mState);

        mState.mStructureChanged = false;

        // onLayoutChildren may have caused client code to disable item animations; re-check
        mState.mRunSimpleAnimations = mState.mRunSimpleAnimations && mItemAnimator != null;
        mState.mLayoutStep = State.STEP_ANIMATIONS;
        onExitLayoutOrScroll();
        stopInterceptRequestLayout(false);
    }

```



## LinearLayoutManager#onLayoutChildren

```java
@Override
    public void onLayoutChildren(RecyclerView.Recycler recycler, RecyclerView.State state) {
        // layout algorithm:
        // 1) by checking children and other variables, find an anchor coordinate and an anchor
        //  item position.
        // 2) fill towards start, stacking from bottom
        // 3) fill towards end, stacking from top
        // 4) scroll to fulfill requirements like stack from bottom.
        // create layout state
        if (DEBUG) {
            Log.d(TAG, "is pre layout:" + state.isPreLayout());
        }
        if (mPendingSavedState != null || mPendingScrollPosition != RecyclerView.NO_POSITION) {
            if (state.getItemCount() == 0) {
                removeAndRecycleAllViews(recycler);
                return;
            }
        }
        if (mPendingSavedState != null && mPendingSavedState.hasValidAnchor()) {
            mPendingScrollPosition = mPendingSavedState.mAnchorPosition;
        }

        ensureLayoutState();
        mLayoutState.mRecycle = false;
        // resolve layout direction
        resolveShouldLayoutReverse();

        final View focused = getFocusedChild();
        if (!mAnchorInfo.mValid || mPendingScrollPosition != RecyclerView.NO_POSITION
                || mPendingSavedState != null) {
            mAnchorInfo.reset();
            mAnchorInfo.mLayoutFromEnd = mShouldReverseLayout ^ mStackFromEnd;
            // calculate anchor position and coordinate
            updateAnchorInfoForLayout(recycler, state, mAnchorInfo);
            mAnchorInfo.mValid = true;
        } else if (focused != null && (mOrientationHelper.getDecoratedStart(focused)
                >= mOrientationHelper.getEndAfterPadding()
                || mOrientationHelper.getDecoratedEnd(focused)
                <= mOrientationHelper.getStartAfterPadding())) {
            // This case relates to when the anchor child is the focused view and due to layout
            // shrinking the focused view fell outside the viewport, e.g. when soft keyboard shows
            // up after tapping an EditText which shrinks RV causing the focused view (The tapped
            // EditText which is the anchor child) to get kicked out of the screen. Will update the
            // anchor coordinate in order to make sure that the focused view is laid out. Otherwise,
            // the available space in layoutState will be calculated as negative preventing the
            // focused view from being laid out in fill.
            // Note that we won't update the anchor position between layout passes (refer to
            // TestResizingRelayoutWithAutoMeasure), which happens if we were to call
            // updateAnchorInfoForLayout for an anchor that's not the focused view (e.g. a reference
            // child which can change between layout passes).
            mAnchorInfo.assignFromViewAndKeepVisibleRect(focused, getPosition(focused));
        }
        if (DEBUG) {
            Log.d(TAG, "Anchor info:" + mAnchorInfo);
        }

        // LLM may decide to layout items for "extra" pixels to account for scrolling target,
        // caching or predictive animations.

        mLayoutState.mLayoutDirection = mLayoutState.mLastScrollDelta >= 0
                ? LayoutState.LAYOUT_END : LayoutState.LAYOUT_START;
        mReusableIntPair[0] = 0;
        mReusableIntPair[1] = 0;
        calculateExtraLayoutSpace(state, mReusableIntPair);
        int extraForStart = Math.max(0, mReusableIntPair[0])
                + mOrientationHelper.getStartAfterPadding();
        int extraForEnd = Math.max(0, mReusableIntPair[1])
                + mOrientationHelper.getEndPadding();
        if (state.isPreLayout() && mPendingScrollPosition != RecyclerView.NO_POSITION
                && mPendingScrollPositionOffset != INVALID_OFFSET) {
            // if the child is visible and we are going to move it around, we should layout
            // extra items in the opposite direction to make sure new items animate nicely
            // instead of just fading in
            final View existing = findViewByPosition(mPendingScrollPosition);
            if (existing != null) {
                final int current;
                final int upcomingOffset;
                if (mShouldReverseLayout) {
                    current = mOrientationHelper.getEndAfterPadding()
                            - mOrientationHelper.getDecoratedEnd(existing);
                    upcomingOffset = current - mPendingScrollPositionOffset;
                } else {
                    current = mOrientationHelper.getDecoratedStart(existing)
                            - mOrientationHelper.getStartAfterPadding();
                    upcomingOffset = mPendingScrollPositionOffset - current;
                }
                if (upcomingOffset > 0) {
                    extraForStart += upcomingOffset;
                } else {
                    extraForEnd -= upcomingOffset;
                }
            }
        }
        int startOffset;
        int endOffset;
        final int firstLayoutDirection;
        if (mAnchorInfo.mLayoutFromEnd) {
            firstLayoutDirection = mShouldReverseLayout ? LayoutState.ITEM_DIRECTION_TAIL
                    : LayoutState.ITEM_DIRECTION_HEAD;
        } else {
            firstLayoutDirection = mShouldReverseLayout ? LayoutState.ITEM_DIRECTION_HEAD
                    : LayoutState.ITEM_DIRECTION_TAIL;
        }

        onAnchorReady(recycler, state, mAnchorInfo, firstLayoutDirection);
        detachAndScrapAttachedViews(recycler);
        mLayoutState.mInfinite = resolveIsInfinite();
        mLayoutState.mIsPreLayout = state.isPreLayout();
        // noRecycleSpace not needed: recycling doesn't happen in below's fill
        // invocations because mScrollingOffset is set to SCROLLING_OFFSET_NaN
        mLayoutState.mNoRecycleSpace = 0;
        if (mAnchorInfo.mLayoutFromEnd) {
            // fill towards start
            updateLayoutStateToFillStart(mAnchorInfo);
            mLayoutState.mExtraFillSpace = extraForStart;
            fill(recycler, mLayoutState, state, false);
            startOffset = mLayoutState.mOffset;
            final int firstElement = mLayoutState.mCurrentPosition;
            if (mLayoutState.mAvailable > 0) {
                extraForEnd += mLayoutState.mAvailable;
            }
            // fill towards end
            updateLayoutStateToFillEnd(mAnchorInfo);
            mLayoutState.mExtraFillSpace = extraForEnd;
            mLayoutState.mCurrentPosition += mLayoutState.mItemDirection;
            fill(recycler, mLayoutState, state, false);
            endOffset = mLayoutState.mOffset;

            if (mLayoutState.mAvailable > 0) {
                // end could not consume all. add more items towards start
                extraForStart = mLayoutState.mAvailable;
                updateLayoutStateToFillStart(firstElement, startOffset);
                mLayoutState.mExtraFillSpace = extraForStart;
                fill(recycler, mLayoutState, state, false);
                startOffset = mLayoutState.mOffset;
            }
        } else {
            // fill towards end
            updateLayoutStateToFillEnd(mAnchorInfo);
            mLayoutState.mExtraFillSpace = extraForEnd;
            fill(recycler, mLayoutState, state, false);
            endOffset = mLayoutState.mOffset;
            final int lastElement = mLayoutState.mCurrentPosition;
            if (mLayoutState.mAvailable > 0) {
                extraForStart += mLayoutState.mAvailable;
            }
            // fill towards start
            updateLayoutStateToFillStart(mAnchorInfo);
            mLayoutState.mExtraFillSpace = extraForStart;
            mLayoutState.mCurrentPosition += mLayoutState.mItemDirection;
            fill(recycler, mLayoutState, state, false);
            startOffset = mLayoutState.mOffset;

            if (mLayoutState.mAvailable > 0) {
                extraForEnd = mLayoutState.mAvailable;
                // start could not consume all it should. add more items towards end
                updateLayoutStateToFillEnd(lastElement, endOffset);
                mLayoutState.mExtraFillSpace = extraForEnd;
                fill(recycler, mLayoutState, state, false);
                endOffset = mLayoutState.mOffset;
            }
        }

        // changes may cause gaps on the UI, try to fix them.
        // TODO we can probably avoid this if neither stackFromEnd/reverseLayout/RTL values have
        // changed
        if (getChildCount() > 0) {
            // because layout from end may be changed by scroll to position
            // we re-calculate it.
            // find which side we should check for gaps.
            if (mShouldReverseLayout ^ mStackFromEnd) {
                int fixOffset = fixLayoutEndGap(endOffset, recycler, state, true);
                startOffset += fixOffset;
                endOffset += fixOffset;
                fixOffset = fixLayoutStartGap(startOffset, recycler, state, false);
                startOffset += fixOffset;
                endOffset += fixOffset;
            } else {
                int fixOffset = fixLayoutStartGap(startOffset, recycler, state, true);
                startOffset += fixOffset;
                endOffset += fixOffset;
                fixOffset = fixLayoutEndGap(endOffset, recycler, state, false);
                startOffset += fixOffset;
                endOffset += fixOffset;
            }
        }
        layoutForPredictiveAnimations(recycler, state, startOffset, endOffset);
        if (!state.isPreLayout()) {
            mOrientationHelper.onLayoutComplete();
        } else {
            mAnchorInfo.reset();
        }
        mLastStackFromEnd = mStackFromEnd;
        if (DEBUG) {
            validateChildOrder();
        }
    }

```



## LinearLayoutManager#fill

```java
   /**
     * The magic functions :). Fills the given layout, defined by the layoutState. This is fairly
     * independent from the rest of the {@link LinearLayoutManager}
     * and with little change, can be made publicly available as a helper class.
     *
     * @param recycler        Current recycler that is attached to RecyclerView
     * @param layoutState     Configuration on how we should fill out the available space.
     * @param state           Context passed by the RecyclerView to control scroll steps.
     * @param stopOnFocusable If true, filling stops in the first focusable new child
     * @return Number of pixels that it added. Useful for scroll functions.
     */
    int fill(RecyclerView.Recycler recycler, LayoutState layoutState,
            RecyclerView.State state, boolean stopOnFocusable) {
        // max offset we should set is mFastScroll + available
        final int start = layoutState.mAvailable;
        if (layoutState.mScrollingOffset != LayoutState.SCROLLING_OFFSET_NaN) {
            // TODO ugly bug fix. should not happen
            if (layoutState.mAvailable < 0) {
                layoutState.mScrollingOffset += layoutState.mAvailable;
            }
            recycleByLayoutState(recycler, layoutState);
        }
        int remainingSpace = layoutState.mAvailable + layoutState.mExtraFillSpace;
        LayoutChunkResult layoutChunkResult = mLayoutChunkResult;
        while ((layoutState.mInfinite || remainingSpace > 0) && layoutState.hasMore(state)) {
            layoutChunkResult.resetInternal();
            if (RecyclerView.VERBOSE_TRACING) {
                TraceCompat.beginSection("LLM LayoutChunk");
            }
            layoutChunk(recycler, state, layoutState, layoutChunkResult);
            if (RecyclerView.VERBOSE_TRACING) {
                TraceCompat.endSection();
            }
            if (layoutChunkResult.mFinished) {
                break;
            }
            layoutState.mOffset += layoutChunkResult.mConsumed * layoutState.mLayoutDirection;
            /**
             * Consume the available space if:
             * * layoutChunk did not request to be ignored
             * * OR we are laying out scrap children
             * * OR we are not doing pre-layout
             */
            if (!layoutChunkResult.mIgnoreConsumed || layoutState.mScrapList != null
                    || !state.isPreLayout()) {
                layoutState.mAvailable -= layoutChunkResult.mConsumed;
                // we keep a separate remaining space because mAvailable is important for recycling
                remainingSpace -= layoutChunkResult.mConsumed;
            }

            if (layoutState.mScrollingOffset != LayoutState.SCROLLING_OFFSET_NaN) {
                layoutState.mScrollingOffset += layoutChunkResult.mConsumed;
                if (layoutState.mAvailable < 0) {
                    layoutState.mScrollingOffset += layoutState.mAvailable;
                }
                recycleByLayoutState(recycler, layoutState);
            }
            if (stopOnFocusable && layoutChunkResult.mFocusable) {
                break;
            }
        }
        if (DEBUG) {
            validateChildOrder();
        }
        return start - layoutState.mAvailable;
    }
```



## LinearLayoutManager#layoutChunk



```java
  void layoutChunk(RecyclerView.Recycler recycler, RecyclerView.State state,
            LayoutState layoutState, LayoutChunkResult result) {
        View view = layoutState.next(recycler);
        if (view == null) {
            if (DEBUG && layoutState.mScrapList == null) {
                throw new RuntimeException("received null view when unexpected");
            }
            // if we are laying out views in scrap, this may return null which means there is
            // no more items to layout.
            result.mFinished = true;
            return;
        }
        RecyclerView.LayoutParams params = (RecyclerView.LayoutParams) view.getLayoutParams();
        if (layoutState.mScrapList == null) {
            if (mShouldReverseLayout == (layoutState.mLayoutDirection
                    == LayoutState.LAYOUT_START)) {
                addView(view);
            } else {
                addView(view, 0);
            }
        } else {
            if (mShouldReverseLayout == (layoutState.mLayoutDirection
                    == LayoutState.LAYOUT_START)) {
                addDisappearingView(view);
            } else {
                addDisappearingView(view, 0);
            }
        }
        measureChildWithMargins(view, 0, 0);
        result.mConsumed = mOrientationHelper.getDecoratedMeasurement(view);
        int left, top, right, bottom;
        if (mOrientation == VERTICAL) {
            if (isLayoutRTL()) {
                right = getWidth() - getPaddingRight();
                left = right - mOrientationHelper.getDecoratedMeasurementInOther(view);
            } else {
                left = getPaddingLeft();
                right = left + mOrientationHelper.getDecoratedMeasurementInOther(view);
            }
            if (layoutState.mLayoutDirection == LayoutState.LAYOUT_START) {
                bottom = layoutState.mOffset;
                top = layoutState.mOffset - result.mConsumed;
            } else {
                top = layoutState.mOffset;
                bottom = layoutState.mOffset + result.mConsumed;
            }
        } else {
            top = getPaddingTop();
            bottom = top + mOrientationHelper.getDecoratedMeasurementInOther(view);

            if (layoutState.mLayoutDirection == LayoutState.LAYOUT_START) {
                right = layoutState.mOffset;
                left = layoutState.mOffset - result.mConsumed;
            } else {
                left = layoutState.mOffset;
                right = layoutState.mOffset + result.mConsumed;
            }
        }
        // We calculate everything with View's bounding box (which includes decor and margins)
        // To calculate correct layout position, we subtract margins.
        layoutDecoratedWithMargins(view, left, top, right, bottom);
        if (DEBUG) {
            Log.d(TAG, "laid out child at position " + getPosition(view) + ", with l:"
                    + (left + params.leftMargin) + ", t:" + (top + params.topMargin) + ", r:"
                    + (right - params.rightMargin) + ", b:" + (bottom - params.bottomMargin));
        }
        // Consume the available space if the view is not removed OR changed
        if (params.isItemRemoved() || params.isItemChanged()) {
            result.mIgnoreConsumed = true;
        }
        result.mFocusable = view.hasFocusable();
    }

```



## LinearLayoutManager.LayoutState#next

```java
     /**
         * Gets the view for the next element that we should layout.
         * Also updates current item index to the next item, based on {@link #mItemDirection}
         *
         * @return The next element that we should layout.
         */
        View next(RecyclerView.Recycler recycler) {
            if (mScrapList != null) {
                return nextViewFromScrapList();
            }
            final View view = recycler.getViewForPosition(mCurrentPosition);
            mCurrentPosition += mItemDirection;
            return view;
        }
```





## RecyclerView$Recycler#getViewForPosition



```java
 /**
         * Obtain a view initialized for the given position.
         *
         * This method should be used by {@link LayoutManager} implementations to obtain
         * views to represent data from an {@link Adapter}.
         * <p>
         * The Recycler may reuse a scrap or detached view from a shared pool if one is
         * available for the correct view type. If the adapter has not indicated that the
         * data at the given position has changed, the Recycler will attempt to hand back
         * a scrap view that was previously initialized for that data without rebinding.
         *
         * @param position Position to obtain a view for
         * @return A view representing the data at <code>position</code> from <code>adapter</code>
         */
        @NonNull
        public View getViewForPosition(int position) {
            return getViewForPosition(position, false);
        }

        View getViewForPosition(int position, boolean dryRun) {
            return tryGetViewHolderForPositionByDeadline(position, dryRun, FOREVER_NS).itemView;
        }
```





## RecyclerView$Recycler#tryGetViewHolderForPositionByDeadline

```java
 
final ArrayList<ViewHolder> mAttachedScrap = new ArrayList<>();
ArrayList<ViewHolder> mChangedScrap = null;

final ArrayList<ViewHolder> mCachedViews = new ArrayList<ViewHolder>();

private final List<ViewHolder>
                mUnmodifiableAttachedScrap = Collections.unmodifiableList(mAttachedScrap);

RecycledViewPool mRecyclerPool;

     /**
         * Mark an attached view as scrap.
         *
         * <p>"Scrap" views are still attached to their parent RecyclerView but are eligible
         * for rebinding and reuse. Requests for a view for a given position may return a
         * reused or rebound scrap view instance.</p>
         *
         * @param view View to scrap
         */
        void scrapView(View view) {
            final ViewHolder holder = getChildViewHolderInt(view);
            if (holder.hasAnyOfTheFlags(ViewHolder.FLAG_REMOVED | ViewHolder.FLAG_INVALID)
                    || !holder.isUpdated() || canReuseUpdatedViewHolder(holder)) {
                if (holder.isInvalid() && !holder.isRemoved() && !mAdapter.hasStableIds()) {
                    throw new IllegalArgumentException("Called scrap view with an invalid view."
                            + " Invalid views cannot be reused from scrap, they should rebound from"
                            + " recycler pool." + exceptionLabel());
                }
                holder.setScrapContainer(this, false);
                mAttachedScrap.add(holder);
            } else {
                // mChangedScrap 实例化
                if (mChangedScrap == null) {
                    mChangedScrap = new ArrayList<ViewHolder>();
                }
                holder.setScrapContainer(this, true);
                mChangedScrap.add(holder);
            }
        }


/**
         * Attempts to get the ViewHolder for the given position, either from the Recycler scrap,
         * cache, the RecycledViewPool, or creating it directly.
         * <p>
         * If a deadlineNs other than {@link #FOREVER_NS} is passed, this method early return
         * rather than constructing or binding a ViewHolder if it doesn't think it has time.
         * If a ViewHolder must be constructed and not enough time remains, null is returned. If a
         * ViewHolder is aquired and must be bound but not enough time remains, an unbound holder is
         * returned. Use {@link ViewHolder#isBound()} on the returned object to check for this.
         *
         * @param position   Position of ViewHolder to be returned.
         * @param dryRun     True if the ViewHolder should not be removed from scrap/cache/
         * @param deadlineNs Time, relative to getNanoTime(), by which bind/create work should
         *                   complete. If FOREVER_NS is passed, this method will not fail to
         *                   create/bind the holder if needed.
         * @return ViewHolder for requested position
         */
        @Nullable
        ViewHolder tryGetViewHolderForPositionByDeadline(int position,
                boolean dryRun, long deadlineNs) {
            if (position < 0 || position >= mState.getItemCount()) {
                throw new IndexOutOfBoundsException("Invalid item position " + position
                        + "(" + position + "). Item count:" + mState.getItemCount()
                        + exceptionLabel());
            }
            boolean fromScrapOrHiddenOrCache = false;
            ViewHolder holder = null;
            
            // 0) If there is a changed scrap, try to find from there
            if (mState.isPreLayout()) {
                //从 mChangeScrap 里查找
                holder = getChangedScrapViewForPosition(position);
                fromScrapOrHiddenOrCache = holder != null;
            }
            // 1) Find by position from scrap/hidden list/cache
            if (holder == null) {
                //通过 position 分别在 mAttachScrap，隐藏的 View，mCachedViews 里查找
                holder = getScrapOrHiddenOrCachedHolderForPosition(position, dryRun);
                if (holder != null) {
                    if (!validateViewHolderForOffsetPosition(holder)) {
                        // recycle holder (and unscrap if relevant) since it can't be used
                        if (!dryRun) {
                            // we would like to recycle this but need to make sure it is not used by
                            // animation logic etc.
                            holder.addFlags(ViewHolder.FLAG_INVALID);
                            if (holder.isScrap()) {
                                removeDetachedView(holder.itemView, false);
                                holder.unScrap();
                            } else if (holder.wasReturnedFromScrap()) {
                                holder.clearReturnedFromScrapFlag();
                            }
                            recycleViewHolderInternal(holder);
                        }
                        holder = null;
                    } else {
                        fromScrapOrHiddenOrCache = true;
                    }
                }
            }
            if (holder == null) {
                final int offsetPosition = mAdapterHelper.findPositionOffset(position);
                if (offsetPosition < 0 || offsetPosition >= mAdapter.getItemCount()) {
                    throw new IndexOutOfBoundsException("Inconsistency detected. Invalid item "
                            + "position " + position + "(offset:" + offsetPosition + ")."
                            + "state:" + mState.getItemCount() + exceptionLabel());
                }

                final int type = mAdapter.getItemViewType(offsetPosition);
                // 2) Find from scrap/cache via stable ids, if exists
                if (mAdapter.hasStableIds()) {
                    //通过 stableid 在 mAttachScrap， mCachedViews 里查找
                    holder = getScrapOrCachedViewForId(mAdapter.getItemId(offsetPosition),
                            type, dryRun);
                    if (holder != null) {
                        // update position
                        holder.mPosition = offsetPosition;
                        fromScrapOrHiddenOrCache = true;
                    }
                }
                if (holder == null && mViewCacheExtension != null) {
                    // We are NOT sending the offsetPosition because LayoutManager does not
                    // know it.
                    final View view = mViewCacheExtension
                            .getViewForPositionAndType(this, position, type);
                    if (view != null) {
                        holder = getChildViewHolder(view);
                        if (holder == null) {
                            throw new IllegalArgumentException("getViewForPositionAndType returned"
                                    + " a view which does not have a ViewHolder"
                                    + exceptionLabel());
                        } else if (holder.shouldIgnore()) {
                            throw new IllegalArgumentException("getViewForPositionAndType returned"
                                    + " a view that is ignored. You must call stopIgnoring before"
                                    + " returning this view." + exceptionLabel());
                        }
                    }
                }
                if (holder == null) { // fallback to pool
                    if (DEBUG) {
                        Log.d(TAG, "tryGetViewHolderForPositionByDeadline("
                                + position + ") fetching from shared pool");
                    }
                    //在 RecycledViewPool 缓存池里查找
                    holder = getRecycledViewPool().getRecycledView(type);
                    if (holder != null) {
                        holder.resetInternal();
                        if (FORCE_INVALIDATE_DISPLAY_LIST) {
                            invalidateDisplayListInt(holder);
                        }
                    }
                }
                if (holder == null) {
                    long start = getNanoTime();
                    if (deadlineNs != FOREVER_NS
                            && !mRecyclerPool.willCreateInTime(type, start, deadlineNs)) {
                        // abort - we have a deadline we can't meet
                        return null;
                    }
                    //无缓存，那么进行创建
                    holder = mAdapter.createViewHolder(RecyclerView.this, type);
                    if (ALLOW_THREAD_GAP_WORK) {
                        // only bother finding nested RV if prefetching
                        RecyclerView innerView = findNestedRecyclerView(holder.itemView);
                        if (innerView != null) {
                            holder.mNestedRecyclerView = new WeakReference<>(innerView);
                        }
                    }

                    long end = getNanoTime();
                    mRecyclerPool.factorInCreateTime(type, end - start);
                    if (DEBUG) {
                        Log.d(TAG, "tryGetViewHolderForPositionByDeadline created new ViewHolder");
                    }
                }
            }

            // This is very ugly but the only place we can grab this information
            // before the View is rebound and returned to the LayoutManager for post layout ops.
            // We don't need this in pre-layout since the VH is not updated by the LM.
            if (fromScrapOrHiddenOrCache && !mState.isPreLayout() && holder
                    .hasAnyOfTheFlags(ViewHolder.FLAG_BOUNCED_FROM_HIDDEN_LIST)) {
                holder.setFlags(0, ViewHolder.FLAG_BOUNCED_FROM_HIDDEN_LIST);
                if (mState.mRunSimpleAnimations) {
                    int changeFlags = ItemAnimator
                            .buildAdapterChangeFlagsForAnimations(holder);
                    changeFlags |= ItemAnimator.FLAG_APPEARED_IN_PRE_LAYOUT;
                    final ItemHolderInfo info = mItemAnimator.recordPreLayoutInformation(mState,
                            holder, changeFlags, holder.getUnmodifiedPayloads());
                    recordAnimationInfoIfBouncedHiddenView(holder, info);
                }
            }

            boolean bound = false;
            if (mState.isPreLayout() && holder.isBound()) {
                // do not update unless we absolutely have to.
                holder.mPreLayoutPosition = position;
            } else if (!holder.isBound() || holder.needsUpdate() || holder.isInvalid()) {
                if (DEBUG && holder.isRemoved()) {
                    throw new IllegalStateException("Removed holder should be bound and it should"
                            + " come here only in pre-layout. Holder: " + holder
                            + exceptionLabel());
                }
                final int offsetPosition = mAdapterHelper.findPositionOffset(position);
                bound = tryBindViewHolderByDeadline(holder, offsetPosition, position, deadlineNs);
            }

            final ViewGroup.LayoutParams lp = holder.itemView.getLayoutParams();
            final LayoutParams rvLayoutParams;
            if (lp == null) {
                rvLayoutParams = (LayoutParams) generateDefaultLayoutParams();
                holder.itemView.setLayoutParams(rvLayoutParams);
            } else if (!checkLayoutParams(lp)) {
                rvLayoutParams = (LayoutParams) generateLayoutParams(lp);
                holder.itemView.setLayoutParams(rvLayoutParams);
            } else {
                rvLayoutParams = (LayoutParams) lp;
            }
            rvLayoutParams.mViewHolder = holder;
            rvLayoutParams.mPendingInvalidate = fromScrapOrHiddenOrCache && bound;
            return holder;
        }
```



### 1 RecyclerView$Recycler#getChangedScrapViewForPosition



```java
  ViewHolder getChangedScrapViewForPosition(int position) {
            // If pre-layout, check the changed scrap for an exact match.
            final int changedScrapSize;
            if (mChangedScrap == null || (changedScrapSize = mChangedScrap.size()) == 0) {
                return null;
            }
            //通过 LayoutPosition
            // find by position
            for (int i = 0; i < changedScrapSize; i++) {
                final ViewHolder holder = mChangedScrap.get(i);
                if (!holder.wasReturnedFromScrap() && holder.getLayoutPosition() == position) {
                    holder.addFlags(ViewHolder.FLAG_RETURNED_FROM_SCRAP);
                    return holder;
                }
            }
            //通过 ItemId
            // find by id
            if (mAdapter.hasStableIds()) {
                final int offsetPosition = mAdapterHelper.findPositionOffset(position);
                if (offsetPosition > 0 && offsetPosition < mAdapter.getItemCount()) {
                    final long id = mAdapter.getItemId(offsetPosition);
                    for (int i = 0; i < changedScrapSize; i++) {
                        final ViewHolder holder = mChangedScrap.get(i);
                        if (!holder.wasReturnedFromScrap() && holder.getItemId() == id) {
                            holder.addFlags(ViewHolder.FLAG_RETURNED_FROM_SCRAP);
                            return holder;
                        }
                    }
                }
            }
            return null;
        }

```



### 2 RecyclerView$Recycler#getScrapOrHiddenOrCachedHolderForPosition

```java
      /**
         * Returns a view for the position either from attach scrap, hidden children, or cache.
         *
         * @param position Item position
         * @param dryRun   Does a dry run, finds the ViewHolder but does not remove
         * @return a ViewHolder that can be re-used for this position.
         */
        ViewHolder getScrapOrHiddenOrCachedHolderForPosition(int position, boolean dryRun) {
            final int scrapCount = mAttachedScrap.size();

            // Try first for an exact, non-invalid match from scrap.
            for (int i = 0; i < scrapCount; i++) {
                final ViewHolder holder = mAttachedScrap.get(i);
                if (!holder.wasReturnedFromScrap() && holder.getLayoutPosition() == position
                        && !holder.isInvalid() && (mState.mInPreLayout || !holder.isRemoved())) {
                    holder.addFlags(ViewHolder.FLAG_RETURNED_FROM_SCRAP);
                    return holder;
                }
            }

            if (!dryRun) {
                View view = mChildHelper.findHiddenNonRemovedView(position);
                if (view != null) {
                    // This View is good to be used. We just need to unhide, detach and move to the
                    // scrap list.
                    final ViewHolder vh = getChildViewHolderInt(view);
                    mChildHelper.unhide(view);
                    int layoutIndex = mChildHelper.indexOfChild(view);
                    if (layoutIndex == RecyclerView.NO_POSITION) {
                        throw new IllegalStateException("layout index should not be -1 after "
                                + "unhiding a view:" + vh + exceptionLabel());
                    }
                    mChildHelper.detachViewFromParent(layoutIndex);
                    scrapView(view);
                    vh.addFlags(ViewHolder.FLAG_RETURNED_FROM_SCRAP
                            | ViewHolder.FLAG_BOUNCED_FROM_HIDDEN_LIST);
                    return vh;
                }
            }

            // Search in our first-level recycled view cache.
            final int cacheSize = mCachedViews.size();
            for (int i = 0; i < cacheSize; i++) {
                final ViewHolder holder = mCachedViews.get(i);
                // invalid view holders may be in cache if adapter has stable ids as they can be
                // retrieved via getScrapOrCachedViewForId
                if (!holder.isInvalid() && holder.getLayoutPosition() == position
                        && !holder.isAttachedToTransitionOverlay()) {
                    if (!dryRun) {
                        mCachedViews.remove(i);
                    }
                    if (DEBUG) {
                        Log.d(TAG, "getScrapOrHiddenOrCachedHolderForPosition(" + position
                                + ") found match in cache: " + holder);
                    }
                    return holder;
                }
            }
            return null;
        }

```

#### 2.1 androidx.recyclerview.widget.ChildHelper#findHiddenNonRemovedView

```java
   View findHiddenNonRemovedView(int position) {
        final int count = mHiddenViews.size();
        for (int i = 0; i < count; i++) {
            final View view = mHiddenViews.get(i);
            RecyclerView.ViewHolder holder = mCallback.getChildViewHolder(view);
            if (holder.getLayoutPosition() == position
                    && !holder.isInvalid()
                    && !holder.isRemoved()) {
                return view;
            }
        }
        return null;
    }
       /**
     * Moves a child view from hidden list to regular list.
     * Calling this method should probably be followed by a detach, otherwise, it will suddenly
     * show up in LayoutManager's children list.
     *
     * @param view The hidden View to unhide
     */
    void unhide(View view) {
        final int offset = mCallback.indexOfChild(view);
        if (offset < 0) {
            throw new IllegalArgumentException("view is not a child, cannot hide " + view);
        }
        if (!mBucket.get(offset)) {
            throw new RuntimeException("trying to unhide a view that was not hidden" + view);
        }
        mBucket.clear(offset);
        unhideViewInternal(view);
    }
      /**
     * Unmarks a child view as hidden.
     *
     * @param child  View to hide.
     */
    private boolean unhideViewInternal(View child) {
        if (mHiddenViews.remove(child)) {
            mCallback.onLeftHiddenState(child);
            return true;
        } else {
            return false;
        }
    }
```

### 3 RecyclerView$Recycler#getScrapOrCachedViewForId

```java
  ViewHolder getScrapOrCachedViewForId(long id, int type, boolean dryRun) {
            // Look in our attached views first
            final int count = mAttachedScrap.size();
            for (int i = count - 1; i >= 0; i--) {
                final ViewHolder holder = mAttachedScrap.get(i);
                if (holder.getItemId() == id && !holder.wasReturnedFromScrap()) {
                    if (type == holder.getItemViewType()) {
                        holder.addFlags(ViewHolder.FLAG_RETURNED_FROM_SCRAP);
                        if (holder.isRemoved()) {
                            // this might be valid in two cases:
                            // > item is removed but we are in pre-layout pass
                            // >> do nothing. return as is. make sure we don't rebind
                            // > item is removed then added to another position and we are in
                            // post layout.
                            // >> remove removed and invalid flags, add update flag to rebind
                            // because item was invisible to us and we don't know what happened in
                            // between.
                            if (!mState.isPreLayout()) {
                                holder.setFlags(ViewHolder.FLAG_UPDATE, ViewHolder.FLAG_UPDATE
                                        | ViewHolder.FLAG_INVALID | ViewHolder.FLAG_REMOVED);
                            }
                        }
                        return holder;
                    } else if (!dryRun) {
                        // if we are running animations, it is actually better to keep it in scrap
                        // but this would force layout manager to lay it out which would be bad.
                        // Recycle this scrap. Type mismatch.
                        mAttachedScrap.remove(i);
                        removeDetachedView(holder.itemView, false);
                        quickRecycleScrapView(holder.itemView);
                    }
                }
            }

            // Search the first-level cache
            final int cacheSize = mCachedViews.size();
            for (int i = cacheSize - 1; i >= 0; i--) {
                final ViewHolder holder = mCachedViews.get(i);
                if (holder.getItemId() == id && !holder.isAttachedToTransitionOverlay()) {
                    if (type == holder.getItemViewType()) {
                        if (!dryRun) {
                            mCachedViews.remove(i);
                        }
                        return holder;
                    } else if (!dryRun) {
                        recycleCachedViewAt(i);
                        return null;
                    }
                }
            }
            return null;
        }
```



### 4 RecyclerView$ViewCacheExtension

```java
  /**
     * ViewCacheExtension is a helper class to provide an additional layer of view caching that can
     * be controlled by the developer.
     * <p>
     * When {@link Recycler#getViewForPosition(int)} is called, Recycler checks attached scrap and
     * first level cache to find a matching View. If it cannot find a suitable View, Recycler will
     * call the {@link #getViewForPositionAndType(Recycler, int, int)} before checking
     * {@link RecycledViewPool}.
     * <p>
     * Note that, Recycler never sends Views to this method to be cached. It is developers
     * responsibility to decide whether they want to keep their Views in this custom cache or let
     * the default recycling policy handle it.
     */
    public abstract static class ViewCacheExtension {

        /**
         * Returns a View that can be binded to the given Adapter position.
         * <p>
         * This method should <b>not</b> create a new View. Instead, it is expected to return
         * an already created View that can be re-used for the given type and position.
         * If the View is marked as ignored, it should first call
         * {@link LayoutManager#stopIgnoringView(View)} before returning the View.
         * <p>
         * RecyclerView will re-bind the returned View to the position if necessary.
         *
         * @param recycler The Recycler that can be used to bind the View
         * @param position The adapter position
         * @param type     The type of the View, defined by adapter
         * @return A View that is bound to the given position or NULL if there is no View to re-use
         * @see LayoutManager#ignoreView(View)
         */
        @Nullable
        public abstract View getViewForPositionAndType(@NonNull Recycler recycler, int position,
                int type);
    }
```





### 5 RecyclerView$RecycledViewPool#getRecycledView

```java
    /**
         * Acquire a ViewHolder of the specified type from the pool, or {@code null} if none are
         * present.
         *
         * @param viewType ViewHolder type.
         * @return ViewHolder of the specified type acquired from the pool, or {@code null} if none
         * are present.
         */
        @Nullable
        public ViewHolder getRecycledView(int viewType) {
            final ScrapData scrapData = mScrap.get(viewType);
            if (scrapData != null && !scrapData.mScrapHeap.isEmpty()) {
                final ArrayList<ViewHolder> scrapHeap = scrapData.mScrapHeap;
                for (int i = scrapHeap.size() - 1; i >= 0; i--) {
                    if (!scrapHeap.get(i).isAttachedToTransitionOverlay()) {
                        return scrapHeap.remove(i);
                    }
                }
            }
            return null;
        }
```



## RecyclerView$Adapter#createViewHolder

```java
 /**
         * This method calls {@link #onCreateViewHolder(ViewGroup, int)} to create a new
         * {@link ViewHolder} and initializes some private fields to be used by RecyclerView.
         *
         * @see #onCreateViewHolder(ViewGroup, int)
         */
        @NonNull
        public final VH createViewHolder(@NonNull ViewGroup parent, int viewType) {
            try {
                TraceCompat.beginSection(TRACE_CREATE_VIEW_TAG);
                //调用抽象方法 RecyclerView.Adapter#onCreateViewHolder
                final VH holder = onCreateViewHolder(parent, viewType);
                if (holder.itemView.getParent() != null) {
                    throw new IllegalStateException("ViewHolder views must not be attached when"
                            + " created. Ensure that you are not passing 'true' to the attachToRoot"
                            + " parameter of LayoutInflater.inflate(..., boolean attachToRoot)");
                }
                holder.mItemViewType = viewType;
                return holder;
            } finally {
                TraceCompat.endSection();
            }
        }
```















## 缓存复用

1. 从 mChangedScrap 里面查找
2. 通过 position 从 mAttachedScrap 和 隐藏的 View 和 mCachedViews 里面查找
3. 通过 stableid 从 mAttachedScrap 和 隐藏的 View 和 mCachedViews 里面查找
4. 从用户自定义的 ViewCacheExtension 中复用机制里面查找
5. 通过 viewtype 从回收缓存池里面查找



## 缓存回收

1. 通过 viewholder 状态判断是否无效并且是否没移除掉是否没有 stableid ,如果都是就进一步判断，否则根据 viewholder 上次状态来加入 mAttachedViews 还是 mChangedViews 里面
2. 接着就是判断 mCachedViews 是否满了来判断是否加入 mCachedViews，如果没满就加入 mCachedViews，满了的话就加入到 回收池里面 RecyclerViewPool 。