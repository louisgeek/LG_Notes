> 源码基于 Android 11 API 30

## ListView 简介

- 继承自 AbsListView ，它继承自 AdapterView<ListAdapter>，它又继承自 ViewGroup





## ListView 特点

- android:divider

- addHeaderView、addFooterView

- setOnItemClickListener、setOnItemLongClickListener

- 未实现 ViewHolder 机制，需要自行实现





## Adapter 适配器

- ListView 和数据源之间的桥梁







## ViewHolder

- 主要是用于缓存 findViewById 的 View

```java
	@Override
    public int getViewTypeCount() {
        //super 默认返回 1
        return super.getViewTypeCount();
    }

    @Override
    public int getItemViewType(int position) {
        //super 默认返回 0
        return super.getItemViewType(position);
    }

	@Override
    public View getView(final int position, View convertView, ViewGroup parent) {
        ViewHolder viewHolder;
        if (convertView == null) {
            convertView = LayoutInflater.from(parent.getContext()).inflate(R.layout.item_list_work, parent, false);
            viewHolder = new ViewHolder();
            //查找 view
            viewHolder.id_tv_title = convertView.findViewById(R.id.id_tv_title);
            viewHolder.id_tv_subtitle = convertView.findViewById(R.id.id_tv_subtitle);
            viewHolder.id_tv_work = convertView.findViewById(R.id.id_tv_work);
            //
            convertView.setTag(viewHolder);
        } else {
            viewHolder = (ViewHolder) convertView.getTag();
        }
        Work work = mWorkList.get(position);
        viewHolder.id_tv_title.setText(work.bed);
        viewHolder.id_tv_subtitle.setText(work.patientName);
        viewHolder.id_tv_work.setText(work.workTitle);
        //
        return convertView;
    }
    //
    private static class ViewHolder {
        private TextView id_tv_title;
        private TextView id_tv_subtitle;
        private TextView id_tv_work;
    }
```





## measure





## layout

- 子类不要直接覆盖 onLayout 方法，应该重写 layoutChildren 方法进行处理



#### AbsListView#onLayout

```java
	final RecycleBin mRecycler = new RecycleBin();   
	/**
     * Subclasses should NOT override this method but
     *  {@link #layoutChildren()} instead.
     */
    @Override
    protected void onLayout(boolean changed, int l, int t, int r, int b) {
        super.onLayout(changed, l, t, r, b);

        mInLayout = true;

        final int childCount = getChildCount();
        if (changed) {
            for (int i = 0; i < childCount; i++) {
                getChildAt(i).forceLayout();
            }
            //RecycleBin 
            mRecycler.markChildrenDirty();
        }
        //空方法，子类自行实现
		//Subclasses must override this method to layout their children.
        layoutChildren();

        mOverscrollMax = (b - t) / OVERSCROLL_LIMIT_DIVISOR;

        // TODO: Move somewhere sane. This doesn't belong in onLayout().
        if (mFastScroll != null) {
            mFastScroll.onItemCountChanged(getChildCount(), mItemCount);
        }
        mInLayout = false;
    }
```

- 调用 layoutChildren，空方法，需要子类自行实现





#### AbsListView$RecycleBin 机制

- ActiveView 可见 View，
- ScrapView 不可见 View，全部离开屏幕的 View

当 item 出现在屏幕时候，会从 RecycleBin 里获取 view 缓存，同样如果完全消失出屏幕后，把 view 缓存到 RecycleBin 里



##### RecycleBin#setViewTypeCount

```java
/**
 * Unsorted views that can be used by the adapter as a convert view.
 */
// ViewType-ScrapViewList  
private ArrayList<View>[] mScrapViews;
//item 样式类型数量
private int mViewTypeCount;

private ArrayList<View> mCurrentScrap;

private ArrayList<View> mSkippedScrap;
        
public void setViewTypeCount(int viewTypeCount) {
            if (viewTypeCount < 1) {
                throw new IllegalArgumentException("Can't have a viewTypeCount < 1");
            }
    		//初始化集合数组
            //noinspection unchecked
            ArrayList<View>[] scrapViews = new ArrayList[viewTypeCount];
            for (int i = 0; i < viewTypeCount; i++) {
                //为数组里的每个集合初始化赋值
                scrapViews[i] = new ArrayList<View>();
            }
            mViewTypeCount = viewTypeCount;
            mCurrentScrap = scrapViews[0];
            mScrapViews = scrapViews;
}
```





##### RecycleBin#fillActiveViews、RecycleBin#getActiveView

- 活跃的 View，可见的 View

```java
/**
         * The position of the first view stored in mActiveViews.
         */
        private int mFirstActivePosition;
        /**
         * Views that were on screen at the start of layout. This array is populated at the start of
         * layout, and at the end of layout all view in mActiveViews are moved to mScrapViews.
         * Views in mActiveViews represent a contiguous range of Views, with position of the first
         * view store in mFirstActivePosition.
         */
        private View[] mActiveViews = new View[0];

  /**
         * Fill ActiveViews with all of the children of the AbsListView.
         *
         * @param childCount The minimum number of views mActiveViews should hold
         * @param firstActivePosition The position of the first view that will be stored in
         *        mActiveViews
         */
        void fillActiveViews(int childCount, int firstActivePosition) {
            if (mActiveViews.length < childCount) {
                mActiveViews = new View[childCount];
            }
            mFirstActivePosition = firstActivePosition;

            //noinspection MismatchedReadAndWriteOfArray
            final View[] activeViews = mActiveViews;
            for (int i = 0; i < childCount; i++) {
                View child = getChildAt(i);
                AbsListView.LayoutParams lp = (AbsListView.LayoutParams) child.getLayoutParams();
                // Don't put header or footer views into the scrap heap
                if (lp != null && lp.viewType != ITEM_VIEW_TYPE_HEADER_OR_FOOTER) {
                    // Note:  We do place AdapterView.ITEM_VIEW_TYPE_IGNORE in active views.
                    //        However, we will NOT place them into scrap views.
                    activeViews[i] = child;
                    // Remember the position so that setupChild() doesn't reset state.
                    lp.scrappedFromPosition = firstActivePosition + i;
                }
            }
        }

 /**
         * Get the view corresponding to the specified position. The view will be removed from
         * mActiveViews if it is found.
         *
         * @param position The position to look up in mActiveViews
         * @return The view if it is found, null otherwise
         */
        View getActiveView(int position) {
            int index = position - mFirstActivePosition;
            final View[] activeViews = mActiveViews;
            if (index >=0 && index < activeViews.length) {
                final View match = activeViews[index];
                activeViews[index] = null;
                return match;
            }
            return null;
        }
```



#####  RecycleBin#addScrapView、RecycleBin#getScrapView

- 废弃的 View，不可见的 View

```java
/**
         * Puts a view into the list of scrap views.
         * <p>
         * If the list data hasn't changed or the adapter has stable IDs, views
         * with transient state will be preserved for later retrieval.
         *
         * @param scrap The view to add
         * @param position The view's position within its parent
         */
        void addScrapView(View scrap, int position) {
            final AbsListView.LayoutParams lp = (AbsListView.LayoutParams) scrap.getLayoutParams();
            if (lp == null) {
                // Can't recycle, but we don't know anything about the view.
                // Ignore it completely.
                return;
            }

            lp.scrappedFromPosition = position;

            // Remove but don't scrap header or footer views, or views that
            // should otherwise not be recycled.
            final int viewType = lp.viewType;
            if (!shouldRecycleViewType(viewType)) {
                // Can't recycle. If it's not a header or footer, which have
                // special handling and should be ignored, then skip the scrap
                // heap and we'll fully detach the view later.
                if (viewType != ITEM_VIEW_TYPE_HEADER_OR_FOOTER) {
                    getSkippedScrap().add(scrap);
                }
                return;
            }

            scrap.dispatchStartTemporaryDetach();

            // The the accessibility state of the view may change while temporary
            // detached and we do not allow detached views to fire accessibility
            // events. So we are announcing that the subtree changed giving a chance
            // to clients holding on to a view in this subtree to refresh it.
            notifyViewAccessibilityStateChangedIfNeeded(
                    AccessibilityEvent.CONTENT_CHANGE_TYPE_SUBTREE);

            // Don't scrap views that have transient state.
            final boolean scrapHasTransientState = scrap.hasTransientState();
            if (scrapHasTransientState) {
                if (mAdapter != null && mAdapterHasStableIds) {
                    // If the adapter has stable IDs, we can reuse the view for
                    // the same data.
                    if (mTransientStateViewsById == null) {
                        mTransientStateViewsById = new LongSparseArray<>();
                    }
                    mTransientStateViewsById.put(lp.itemId, scrap);
                } else if (!mDataChanged) {
                    // If the data hasn't changed, we can reuse the views at
                    // their old positions.
                    if (mTransientStateViews == null) {
                        mTransientStateViews = new SparseArray<>();
                    }
                    mTransientStateViews.put(position, scrap);
                } else {
                    // Otherwise, we'll have to remove the view and start over.
                    clearScrapForRebind(scrap);
                    getSkippedScrap().add(scrap);
                }
            } else {
                clearScrapForRebind(scrap);
                if (mViewTypeCount == 1) {
                    mCurrentScrap.add(scrap);
                } else {
                    mScrapViews[viewType].add(scrap);
                }

                if (mRecyclerListener != null) {
                    mRecyclerListener.onMovedToScrapHeap(scrap);
                }
            }
        }

     /**
         * @return A view from the ScrapViews collection. These are unordered.
         */
        View getScrapView(int position) {
            final int whichScrap = mAdapter.getItemViewType(position);
            if (whichScrap < 0) {
                return null;
            }
            if (mViewTypeCount == 1) {
                return retrieveFromScrap(mCurrentScrap, position);
            } else if (whichScrap < mScrapViews.length) {
                return retrieveFromScrap(mScrapViews[whichScrap], position);
            }
            return null;
        }
```



#### ListView#layoutChildren

```java
@Override
    protected void layoutChildren() {
        final boolean blockLayoutRequests = mBlockLayoutRequests;
        if (blockLayoutRequests) {
            return;
        }

        mBlockLayoutRequests = true;

        try {
            super.layoutChildren();

            invalidate();

            if (mAdapter == null) {
                resetList();
                invokeOnItemScrollListener();
                return;
            }

            final int childrenTop = mListPadding.top;
            final int childrenBottom = mBottom - mTop - mListPadding.bottom;
            final int childCount = getChildCount();

            int index = 0;
            int delta = 0;

            View sel;
            View oldSel = null;
            View oldFirst = null;
            View newSel = null;
			//AbsListView#mLayoutMode 初始化默认 LAYOUT_NORMAL
            // Remember stuff we will need down below
            switch (mLayoutMode) {
            case LAYOUT_SET_SELECTION:
                index = mNextSelectedPosition - mFirstPosition;
                if (index >= 0 && index < childCount) {
                    newSel = getChildAt(index);
                }
                break;
            case LAYOUT_FORCE_TOP:
            case LAYOUT_FORCE_BOTTOM:
            case LAYOUT_SPECIFIC:
            case LAYOUT_SYNC:
                break;
            case LAYOUT_MOVE_SELECTION:
            default:
                // LAYOUT_NORMAL 走
                // Remember the previously selected view
                index = mSelectedPosition - mFirstPosition;
                if (index >= 0 && index < childCount) {
                    oldSel = getChildAt(index);
                }

                // Remember the previous first child
                oldFirst = getChildAt(0);

                if (mNextSelectedPosition >= 0) {
                    delta = mNextSelectedPosition - mSelectedPosition;
                }

                // Caution: newSel might be null
                newSel = getChildAt(index + delta);
            }

			// AdapterView#mDataChanged
            boolean dataChanged = mDataChanged;
            if (dataChanged) {
                handleDataChanged();
            }

            // Handle the empty set by removing all views that are visible
            // and calling it a day
            if (mItemCount == 0) {
                resetList();
                invokeOnItemScrollListener();
                return;
            } else if (mItemCount != mAdapter.getCount()) {
                throw new IllegalStateException("The content of the adapter has changed but "
                        + "ListView did not receive a notification. Make sure the content of "
                        + "your adapter is not modified from a background thread, but only from "
                        + "the UI thread. Make sure your adapter calls notifyDataSetChanged() "
                        + "when its content changes. [in ListView(" + getId() + ", " + getClass()
                        + ") with Adapter(" + mAdapter.getClass() + ")]");
            }

            setSelectedPositionInt(mNextSelectedPosition);

            AccessibilityNodeInfo accessibilityFocusLayoutRestoreNode = null;
            View accessibilityFocusLayoutRestoreView = null;
            int accessibilityFocusPosition = INVALID_POSITION;

            // Remember which child, if any, had accessibility focus. This must
            // occur before recycling any views, since that will clear
            // accessibility focus.
            final ViewRootImpl viewRootImpl = getViewRootImpl();
            if (viewRootImpl != null) {
                final View focusHost = viewRootImpl.getAccessibilityFocusedHost();
                if (focusHost != null) {
                    final View focusChild = getAccessibilityFocusedChild(focusHost);
                    if (focusChild != null) {
                        if (!dataChanged || isDirectChildHeaderOrFooter(focusChild)
                                || (focusChild.hasTransientState() && mAdapterHasStableIds)) {
                            // The views won't be changing, so try to maintain
                            // focus on the current host and virtual view.
                            accessibilityFocusLayoutRestoreView = focusHost;
                            accessibilityFocusLayoutRestoreNode = viewRootImpl
                                    .getAccessibilityFocusedVirtualView();
                        }

                        // If all else fails, maintain focus at the same
                        // position.
                        accessibilityFocusPosition = getPositionForView(focusChild);
                    }
                }
            }

            View focusLayoutRestoreDirectChild = null;
            View focusLayoutRestoreView = null;

            // Take focus back to us temporarily to avoid the eventual call to
            // clear focus when removing the focused child below from messing
            // things up when ViewAncestor assigns focus back to someone else.
            final View focusedChild = getFocusedChild();
            if (focusedChild != null) {
                // TODO: in some cases focusedChild.getParent() == null

                // We can remember the focused view to restore after re-layout
                // if the data hasn't changed, or if the focused position is a
                // header or footer.
                if (!dataChanged || isDirectChildHeaderOrFooter(focusedChild)
                        || focusedChild.hasTransientState() || mAdapterHasStableIds) {
                    focusLayoutRestoreDirectChild = focusedChild;
                    // Remember the specific view that had focus.
                    focusLayoutRestoreView = findFocus();
                    if (focusLayoutRestoreView != null) {
                        // Tell it we are going to mess with it.
                        focusLayoutRestoreView.dispatchStartTemporaryDetach();
                    }
                }
                requestFocus();
            }

            // Pull all children into the RecycleBin.
            // These views will be reused if possible
            final int firstPosition = mFirstPosition;
            final RecycleBin recycleBin = mRecycler;
            if (dataChanged) {
                for (int i = 0; i < childCount; i++) {
                    recycleBin.addScrapView(getChildAt(i), firstPosition+i);
                }
            } else {
                recycleBin.fillActiveViews(childCount, firstPosition);
            }

            // Clear out old views
            detachAllViewsFromParent();
            recycleBin.removeSkippedScrap();
			//AbsListView#mLayoutMode 初始化默认 LAYOUT_NORMAL
            switch (mLayoutMode) {
            case LAYOUT_SET_SELECTION:
                if (newSel != null) {
                    sel = fillFromSelection(newSel.getTop(), childrenTop, childrenBottom);
                } else {
                    sel = fillFromMiddle(childrenTop, childrenBottom);
                }
                break;
            case LAYOUT_SYNC:
                sel = fillSpecific(mSyncPosition, mSpecificTop);
                break;
            case LAYOUT_FORCE_BOTTOM:
                sel = fillUp(mItemCount - 1, childrenBottom);
                adjustViewsUpOrDown();
                break;
            case LAYOUT_FORCE_TOP:
                mFirstPosition = 0;
                sel = fillFromTop(childrenTop);
                adjustViewsUpOrDown();
                break;
            case LAYOUT_SPECIFIC:
                final int selectedPosition = reconcileSelectedPosition();
                sel = fillSpecific(selectedPosition, mSpecificTop);
                /**
                 * When ListView is resized, FocusSelector requests an async selection for the
                 * previously focused item to make sure it is still visible. If the item is not
                 * selectable, it won't regain focus so instead we call FocusSelector
                 * to directly request focus on the view after it is visible.
                 */
                if (sel == null && mFocusSelector != null) {
                    final Runnable focusRunnable = mFocusSelector
                            .setupFocusIfValid(selectedPosition);
                    if (focusRunnable != null) {
                        post(focusRunnable);
                    }
                }
                break;
            case LAYOUT_MOVE_SELECTION:
                sel = moveSelection(oldSel, newSel, delta, childrenTop, childrenBottom);
                break;
            default:
                // LAYOUT_NORMAL 走这
                if (childCount == 0) {
                    if (!mStackFromBottom) {
                        final int position = lookForSelectablePosition(0, true);
                        setSelectedPositionInt(position);
                        //调用 ListView#fillFromTop
                        sel = fillFromTop(childrenTop);
                    } else {
                        final int position = lookForSelectablePosition(mItemCount - 1, false);
                        setSelectedPositionInt(position);
                        sel = fillUp(mItemCount - 1, childrenBottom);
                    }
                } else {
                    if (mSelectedPosition >= 0 && mSelectedPosition < mItemCount) {
                        sel = fillSpecific(mSelectedPosition,
                                oldSel == null ? childrenTop : oldSel.getTop());
                    } else if (mFirstPosition < mItemCount) {
                        sel = fillSpecific(mFirstPosition,
                                oldFirst == null ? childrenTop : oldFirst.getTop());
                    } else {
                        sel = fillSpecific(0, childrenTop);
                    }
                }
                break;
            }

            // Flush any cached views that did not get reused above
            recycleBin.scrapActiveViews();

            // remove any header/footer that has been temp detached and not re-attached
            removeUnusedFixedViews(mHeaderViewInfos);
            removeUnusedFixedViews(mFooterViewInfos);

            if (sel != null) {
                // The current selected item should get focus if items are
                // focusable.
                if (mItemsCanFocus && hasFocus() && !sel.hasFocus()) {
                    final boolean focusWasTaken = (sel == focusLayoutRestoreDirectChild &&
                            focusLayoutRestoreView != null &&
                            focusLayoutRestoreView.requestFocus()) || sel.requestFocus();
                    if (!focusWasTaken) {
                        // Selected item didn't take focus, but we still want to
                        // make sure something else outside of the selected view
                        // has focus.
                        final View focused = getFocusedChild();
                        if (focused != null) {
                            focused.clearFocus();
                        }
                        positionSelector(INVALID_POSITION, sel);
                    } else {
                        sel.setSelected(false);
                        mSelectorRect.setEmpty();
                    }
                } else {
                    positionSelector(INVALID_POSITION, sel);
                }
                mSelectedTop = sel.getTop();
            } else {
                final boolean inTouchMode = mTouchMode == TOUCH_MODE_TAP
                        || mTouchMode == TOUCH_MODE_DONE_WAITING;
                if (inTouchMode) {
                    // If the user's finger is down, select the motion position.
                    final View child = getChildAt(mMotionPosition - mFirstPosition);
                    if (child != null) {
                        positionSelector(mMotionPosition, child);
                    }
                } else if (mSelectorPosition != INVALID_POSITION) {
                    // If we had previously positioned the selector somewhere,
                    // put it back there. It might not match up with the data,
                    // but it's transitioning out so it's not a big deal.
                    final View child = getChildAt(mSelectorPosition - mFirstPosition);
                    if (child != null) {
                        positionSelector(mSelectorPosition, child);
                    }
                } else {
                    // Otherwise, clear selection.
                    mSelectedTop = 0;
                    mSelectorRect.setEmpty();
                }

                // Even if there is not selected position, we may need to
                // restore focus (i.e. something focusable in touch mode).
                if (hasFocus() && focusLayoutRestoreView != null) {
                    focusLayoutRestoreView.requestFocus();
                }
            }

            // Attempt to restore accessibility focus, if necessary.
            if (viewRootImpl != null) {
                final View newAccessibilityFocusedView = viewRootImpl.getAccessibilityFocusedHost();
                if (newAccessibilityFocusedView == null) {
                    if (accessibilityFocusLayoutRestoreView != null
                            && accessibilityFocusLayoutRestoreView.isAttachedToWindow()) {
                        final AccessibilityNodeProvider provider =
                                accessibilityFocusLayoutRestoreView.getAccessibilityNodeProvider();
                        if (accessibilityFocusLayoutRestoreNode != null && provider != null) {
                            final int virtualViewId = AccessibilityNodeInfo.getVirtualDescendantId(
                                    accessibilityFocusLayoutRestoreNode.getSourceNodeId());
                            provider.performAction(virtualViewId,
                                    AccessibilityNodeInfo.ACTION_ACCESSIBILITY_FOCUS, null);
                        } else {
                            accessibilityFocusLayoutRestoreView.requestAccessibilityFocus();
                        }
                    } else if (accessibilityFocusPosition != INVALID_POSITION) {
                        // Bound the position within the visible children.
                        final int position = MathUtils.constrain(
                                accessibilityFocusPosition - mFirstPosition, 0,
                                getChildCount() - 1);
                        final View restoreView = getChildAt(position);
                        if (restoreView != null) {
                            restoreView.requestAccessibilityFocus();
                        }
                    }
                }
            }

            // Tell focus view we are done mucking with it, if it is still in
            // our view hierarchy.
            if (focusLayoutRestoreView != null
                    && focusLayoutRestoreView.getWindowToken() != null) {
                focusLayoutRestoreView.dispatchFinishTemporaryDetach();
            }

            mLayoutMode = LAYOUT_NORMAL;
            mDataChanged = false;
            if (mPositionScrollAfterLayout != null) {
                post(mPositionScrollAfterLayout);
                mPositionScrollAfterLayout = null;
            }
            mNeedSync = false;
            setNextSelectedPositionInt(mSelectedPosition);

            updateScrollIndicators();

            if (mItemCount > 0) {
                checkSelectionChanged();
            }

            invokeOnItemScrollListener();
        } finally {
            if (mFocusSelector != null) {
                mFocusSelector.onLayoutComplete();
            }
            if (!blockLayoutRequests) {
                mBlockLayoutRequests = false;
            }
        }
    }
```



#### ListView#fillFromTop、ListView#fillDown

```java
/**
     * Fills the list from top to bottom, starting with mFirstPosition
     *
     * @param nextTop The location where the top of the first item should be
     *        drawn
     *
     * @return The view that is currently selected
     */
    private View fillFromTop(int nextTop) {
        mFirstPosition = Math.min(mFirstPosition, mSelectedPosition);
        mFirstPosition = Math.min(mFirstPosition, mItemCount - 1);
        if (mFirstPosition < 0) {
            mFirstPosition = 0;
        }
        //调用 fillDown
        return fillDown(mFirstPosition, nextTop);
    }
     /**
     * Fills the list from pos down to the end of the list view.
     *
     * @param pos The first position to put in the list
     *
     * @param nextTop The location where the top of the item associated with pos
     *        should be drawn
     *
     * @return The view that is currently selected, if it happens to be in the
     *         range that we draw.
     */
    @UnsupportedAppUsage(maxTargetSdk = Build.VERSION_CODES.P)
    private View fillDown(int pos, int nextTop) {
        View selectedView = null;

        int end = (mBottom - mTop);
        if ((mGroupFlags & CLIP_TO_PADDING_MASK) == CLIP_TO_PADDING_MASK) {
            end -= mListPadding.bottom;
        }

        while (nextTop < end && pos < mItemCount) {
            // is this the selected item?
            boolean selected = pos == mSelectedPosition;
            View child = makeAndAddView(pos, nextTop, true, mListPadding.left, selected);

            nextTop = child.getBottom() + mDividerHeight;
            if (selected) {
                selectedView = child;
            }
            pos++;
        }

        setVisibleRangeHint(mFirstPosition, mFirstPosition + getChildCount() - 1);
        return selectedView;
    }
```





#### AbsListView#obtainView

```java
	ListAdapter mAdapter;    
	/**
     * Gets a view and have it show the data associated with the specified
     * position. This is called when we have already discovered that the view
     * is not available for reuse in the recycle bin. The only choices left are
     * converting an old view or making a new one.
     *
     * @param position the position to display
     * @param outMetadata an array of at least 1 boolean where the first entry
     *                    will be set {@code true} if the view is currently
     *                    attached to the window, {@code false} otherwise (e.g.
     *                    newly-inflated or remained scrap for multiple layout
     *                    passes)
     *
     * @return A view displaying the data associated with the specified position
     */
    View obtainView(int position, boolean[] outMetadata) {
        Trace.traceBegin(Trace.TRACE_TAG_VIEW, "obtainView");

        outMetadata[0] = false;

        // Check whether we have a transient state view. Attempt to re-bind the
        // data and discard the view if we fail.
        final View transientView = mRecycler.getTransientStateView(position);
        if (transientView != null) {
            final LayoutParams params = (LayoutParams) transientView.getLayoutParams();

            // If the view type hasn't changed, attempt to re-bind the data.
            if (params.viewType == mAdapter.getItemViewType(position)) {
                final View updatedView = mAdapter.getView(position, transientView, this);

                // If we failed to re-bind the data, scrap the obtained view.
                if (updatedView != transientView) {
                    setItemViewLayoutParams(updatedView, position);
                    mRecycler.addScrapView(updatedView, position);
                }
            }

            outMetadata[0] = true;

            // Finish the temporary detach started in addScrapView().
            transientView.dispatchFinishTemporaryDetach();
            return transientView;
        }
		//
        final View scrapView = mRecycler.getScrapView(position);
        //！！！就是走适配器的 getView
        final View child = mAdapter.getView(position, scrapView, this);
        if (scrapView != null) {
            if (child != scrapView) {
                // Failed to re-bind the data, return scrap to the heap.
                mRecycler.addScrapView(scrapView, position);
            } else if (child.isTemporarilyDetached()) {
                outMetadata[0] = true;

                // Finish the temporary detach started in addScrapView().
                child.dispatchFinishTemporaryDetach();
            }
        }

        if (mCacheColorHint != 0) {
            child.setDrawingCacheBackgroundColor(mCacheColorHint);
        }

        if (child.getImportantForAccessibility() == IMPORTANT_FOR_ACCESSIBILITY_AUTO) {
            child.setImportantForAccessibility(IMPORTANT_FOR_ACCESSIBILITY_YES);
        }

        setItemViewLayoutParams(child, position);

        if (AccessibilityManager.getInstance(mContext).isEnabled()) {
            if (mAccessibilityDelegate == null) {
                mAccessibilityDelegate = new ListItemAccessibilityDelegate();
            }
            if (child.getAccessibilityDelegate() == null) {
                child.setAccessibilityDelegate(mAccessibilityDelegate);
            }
        }

        Trace.traceEnd(Trace.TRACE_TAG_VIEW);

        return child;
    }
```





## ListView#setAdapter

- BaseAdapter 实现了 ListAdapter 接口

```java
/**
 * Sets the data behind this ListView.
 *
 * The adapter passed to this method may be wrapped by a {@link WrapperListAdapter},
 * depending on the ListView features currently in use. For instance, adding
 * headers and/or footers will cause the adapter to be wrapped.
 *
 * @param adapter The ListAdapter which is responsible for maintaining the
 *        data backing this list and for producing a view to represent an
 *        item in that data set.
 *
 * @see #getAdapter()
 */
@Override
public void setAdapter(ListAdapter adapter) {
    //AbsListView#mAdapter 是一个 ListAdapter
    //AbsListView#mDataSetObserver 是一个 AbsListView#AdapterDataSetObserver
    if (mAdapter != null && mDataSetObserver != null) {
        mAdapter.unregisterDataSetObserver(mDataSetObserver);
    }

    resetList();
    mRecycler.clear();

    if (mHeaderViewInfos.size() > 0|| mFooterViewInfos.size() > 0) {
        mAdapter = wrapHeaderListAdapterInternal(mHeaderViewInfos, mFooterViewInfos, adapter);
    } else {
        mAdapter = adapter;
    }

    mOldSelectedPosition = INVALID_POSITION;
    mOldSelectedRowId = INVALID_ROW_ID;

    // AbsListView#setAdapter will update choice mode states.
    super.setAdapter(adapter);

    if (mAdapter != null) {
        mAreAllItemsSelectable = mAdapter.areAllItemsEnabled();
        mOldItemCount = mItemCount;
        mItemCount = mAdapter.getCount();
        checkFocus();
		//初始化观察者
        mDataSetObserver = new AdapterDataSetObserver();
        mAdapter.registerDataSetObserver(mDataSetObserver);
	
        //AbsListView#mRecycler 是一个 RecycleBin
        mRecycler.setViewTypeCount(mAdapter.getViewTypeCount());

        int position;
        if (mStackFromBottom) {
            position = lookForSelectablePosition(mItemCount - 1, false);
        } else {
            position = lookForSelectablePosition(0, true);
        }
        setSelectedPositionInt(position);
        setNextSelectedPositionInt(position);

        if (mItemCount == 0) {
            // Nothing selected
            checkSelectionChanged();
        }
    } else {
        mAreAllItemsSelectable = true;
        checkFocus();
        // Nothing selected
        checkSelectionChanged();
    }

    requestLayout();
}
```

- AbsListView 中维护了一个 AdapterDataSetObserver 观察者，所以 ListView 可以看成一个观察者

- BaseAdapter 中维护了一个 DataSetObservable 被观察者，所以 Adapter 可以看成一个被观察者

- DataSetObservable 继承 Observable<DataSetObserver>

- 而这个 Observable 里维护了一个存放所有 DataSetObserver 的观察者集合

- AbsListView 维护着一个 ListAdapter

- ListView#setAdapter 给 ListView 设置了这个 Adapter，同时也给上文提到的 Adapter 里的观察者集合中添加了一个 AdapterDataSetObserver 观察者

- 抽象理解就是 ListView 订阅了 Adapter，数据更新后，Adapter 负责通知 ListView 进行视图更新

  

  

### AbsListView#AdapterDataSetObserver

```java
class AdapterDataSetObserver extends AdapterView<ListAdapter>.AdapterDataSetObserver {
        @Override
        public void onChanged() {
            super.onChanged();
            if (mFastScroll != null) {
                mFastScroll.onSectionsChanged();
            }
        }

        @Override
        public void onInvalidated() {
            super.onInvalidated();
            if (mFastScroll != null) {
                mFastScroll.onSectionsChanged();
            }
        }
    }
```



#### AdapterView.AdapterDataSetObserver

```java
 class AdapterDataSetObserver extends DataSetObserver {

        private Parcelable mInstanceState = null;

        @Override
        public void onChanged() {
            mDataChanged = true;
            mOldItemCount = mItemCount;
            mItemCount = getAdapter().getCount();

            // Detect the case where a cursor that was previously invalidated has
            // been repopulated with new data.
            if (AdapterView.this.getAdapter().hasStableIds() && mInstanceState != null
                    && mOldItemCount == 0 && mItemCount > 0) {
                AdapterView.this.onRestoreInstanceState(mInstanceState);
                mInstanceState = null;
            } else {
                rememberSyncState();
            }
            checkFocus();
            requestLayout();
        }

        @Override
        public void onInvalidated() {
            mDataChanged = true;

            if (AdapterView.this.getAdapter().hasStableIds()) {
                // Remember the current state for the case where our hosting activity is being
                // stopped and later restarted
                mInstanceState = AdapterView.this.onSaveInstanceState();
            }

            // Data is invalid so we should reset our state
            mOldItemCount = mItemCount;
            mItemCount = 0;
            mSelectedPosition = INVALID_POSITION;
            mSelectedRowId = INVALID_ROW_ID;
            mNextSelectedPosition = INVALID_POSITION;
            mNextSelectedRowId = INVALID_ROW_ID;
            mNeedSync = false;

            checkFocus();
            requestLayout();
        }

        public void clearSavedState() {
            mInstanceState = null;
        }
    }
```

##### android.database.DataSetObserver

```java
/**
 * Receives call backs when a data set has been changed, or made invalid. The typically data sets
 * that are observed are {@link Cursor}s or {@link android.widget.Adapter}s.
 * DataSetObserver must be implemented by objects which are added to a DataSetObservable.
 */
public abstract class DataSetObserver {
    /**
     * This method is called when the entire data set has changed,
     * most likely through a call to {@link Cursor#requery()} on a {@link Cursor}.
     */
    public void onChanged() {
        // Do nothing
    }

    /**
     * This method is called when the entire data becomes invalid,
     * most likely through a call to {@link Cursor#deactivate()} or {@link Cursor#close()} on a
     * {@link Cursor}.
     */
    public void onInvalidated() {
        // Do nothing
    }
}
```



### BaseAdapter

```java
/**
 * Common base class of common implementation for an {@link Adapter} that can be
 * used in both {@link ListView} (by implementing the specialized
 * {@link ListAdapter} interface) and {@link Spinner} (by implementing the
 * specialized {@link SpinnerAdapter} interface).
 */
public abstract class BaseAdapter implements ListAdapter, SpinnerAdapter {
    @UnsupportedAppUsage
    private final DataSetObservable mDataSetObservable = new DataSetObservable();
    private CharSequence[] mAutofillOptions;

    public boolean hasStableIds() {
        return false;
    }
    
    public void registerDataSetObserver(DataSetObserver observer) {
        mDataSetObservable.registerObserver(observer);
    }

    public void unregisterDataSetObserver(DataSetObserver observer) {
        mDataSetObservable.unregisterObserver(observer);
    }
    
    /**
     * Notifies the attached observers that the underlying data has been changed
     * and any View reflecting the data set should refresh itself.
     */
    public void notifyDataSetChanged() {
        mDataSetObservable.notifyChanged();
    }

    /**
     * Notifies the attached observers that the underlying data is no longer valid
     * or available. Once invoked this adapter is no longer valid and should
     * not report further data set changes.
     */
    public void notifyDataSetInvalidated() {
        mDataSetObservable.notifyInvalidated();
    }

    public boolean areAllItemsEnabled() {
        return true;
    }

    public boolean isEnabled(int position) {
        return true;
    }

    public View getDropDownView(int position, View convertView, ViewGroup parent) {
        return getView(position, convertView, parent);
    }

    public int getItemViewType(int position) {
        return 0;
    }

    public int getViewTypeCount() {
        return 1;
    }
    
    public boolean isEmpty() {
        return getCount() == 0;
    }

    @Override
    public CharSequence[] getAutofillOptions() {
        return mAutofillOptions;
    }

    /**
     * Sets the value returned by {@link #getAutofillOptions()}
     */
    public void setAutofillOptions(@Nullable CharSequence... options) {
        mAutofillOptions = options;
    }
}
```



#### android.database.DataSetObservable

```java
/**
 * A specialization of {@link Observable} for {@link DataSetObserver}
 * that provides methods for sending notifications to a list of
 * {@link DataSetObserver} objects.
 */
public class DataSetObservable extends Observable<DataSetObserver> {
    /**
     * Invokes {@link DataSetObserver#onChanged} on each observer.
     * Called when the contents of the data set have changed.  The recipient
     * will obtain the new contents the next time it queries the data set.
     */
    public void notifyChanged() {
        //ArrayList<DataSetObserver>
        synchronized(mObservers) {
            // since onChanged() is implemented by the app, it could do anything, including
            // removing itself from {@link mObservers} - and that could cause problems if
            // an iterator is used on the ArrayList {@link mObservers}.
            // to avoid such problems, just march thru the list in the reverse order.
            for (int i = mObservers.size() - 1; i >= 0; i--) {
                mObservers.get(i).onChanged();
            }
        }
    }

    /**
     * Invokes {@link DataSetObserver#onInvalidated} on each observer.
     * Called when the data set is no longer valid and cannot be queried again,
     * such as when the data set has been closed.
     */
    public void notifyInvalidated() {
        synchronized (mObservers) {
            for (int i = mObservers.size() - 1; i >= 0; i--) {
                mObservers.get(i).onInvalidated();
            }
        }
    }
}
```



##### android.database.Observable

```java
/**
 * Provides methods for registering or unregistering arbitrary observers in an {@link ArrayList}.
 *
 * This abstract class is intended to be subclassed and specialized to maintain
 * a registry of observers of specific types and dispatch notifications to them.
 *
 * @param T The observer type.
 */
public abstract class Observable<T> {
    /**
     * The list of observers.  An observer can be in the list at most
     * once and will never be null.
     */
    protected final ArrayList<T> mObservers = new ArrayList<T>();

    /**
     * Adds an observer to the list. The observer cannot be null and it must not already
     * be registered.
     * @param observer the observer to register
     * @throws IllegalArgumentException the observer is null
     * @throws IllegalStateException the observer is already registered
     */
    public void registerObserver(T observer) {
        if (observer == null) {
            throw new IllegalArgumentException("The observer is null.");
        }
        synchronized(mObservers) {
            if (mObservers.contains(observer)) {
                throw new IllegalStateException("Observer " + observer + " is already registered.");
            }
            mObservers.add(observer);
        }
    }

    /**
     * Removes a previously registered observer. The observer must not be null and it
     * must already have been registered.
     * @param observer the observer to unregister
     * @throws IllegalArgumentException the observer is null
     * @throws IllegalStateException the observer is not yet registered
     */
    public void unregisterObserver(T observer) {
        if (observer == null) {
            throw new IllegalArgumentException("The observer is null.");
        }
        synchronized(mObservers) {
            int index = mObservers.indexOf(observer);
            if (index == -1) {
                throw new IllegalStateException("Observer " + observer + " was not registered.");
            }
            mObservers.remove(index);
        }
    }

    /**
     * Remove all registered observers.
     */
    public void unregisterAll() {
        synchronized(mObservers) {
            mObservers.clear();
        }
    }
}
```

- BaseAdapter#registerDataSetObserver -> DataSetObservable#registerObserver ->

mObservers#add（ArrayList#add，DataSetObserver 的集合）

- BaseAdapter#unregisterDataSetObserver -> DataSetObservable#unregisterObserver->

mObservers#remove（ArrayList#remove，DataSetObserver 的集合）



#### BaseAdapter#notifyDataSetChanged

// BaseAdapter 调用 notifyDataSetChanged 方法后，直接调用 DataSetObservable 被观察者的 notifyChanged 方法，接着就是遍历观察者集合里所有的 DataSetObserver 执行它的 onChanged 的方法

```
BaseAdapter#notifyDataSetChanged -> DataSetObservable#notifyChanged -> DataSetObserver#onChanged
```

也就是遍历了 AdapterDataSetObserver 进行 onChanged

```
AbsListView#AdapterDataSetObserver#onChanged -> AdapterView#AdapterDataSetObserver#onChanged -> View#requestLayout
```

- 最终调用 View#requestLayout 













为啥 mAdapter.notifyDataSetChanged() 一句话就能完成 ListView 更新？

首要条件 mListView.setAdapter(mAdapter) 方法中生成了 AdapterDataSetObserver 对象赋值给 mDataSetObserver，然后进行了 mAdapter.registerDataSetObserver(mDataSetObserver)，而调用notifyDataSetChanged 方法进行观察者的遍历，然后掉调用其 onChanged 方法，最终调用View#requestLayout 实现



 