# Virtual DOM Reconciliation and Diffing Algorithm

## The Big Picture

The real DOM is slow to update, so React keeps a lightweight copy of it in memory. The Virtual DOM is a tree of lightweight JavaScript objects representing the actual DOM structure.

The update cycle has three steps:

1. **Render**: when state or props change, React builds a new virtual tree.
2. **Diff**: React compares the new tree with the previous one.
3. **Commit**: React applies only the minimal changes to the real DOM.

**Reconciliation** is the whole process. **Diffing** is the comparison step inside it. The goal of diffing is to minimize the number of operations needed to update the real DOM.

## Why It Is a Heuristic

A fully general tree-diff is O(n³). React instead uses heuristics that bring it down to roughly O(n). They rest on two assumptions:

- Elements of different types produce different trees.
- Keys let the developer hint which children stay stable across renders.

## The Diffing Rules

### 1. Different element types: tear down and rebuild

When React sees different element types, it discards the old DOM subtree and builds the new one from scratch. Changing `<div>` to `<section>` unmounts everything inside, including component state.

### 2. Same DOM element type: update in place

React keeps the same underlying DOM node and only updates the attributes that changed, then recurses into the children.

```jsx
<div className="a" title="x" />   // old
<div className="b" title="x" />   // new -> only className is patched
```

### 3. Same component type: keep the instance

React does not discard or replace the existing component. It updates the props and re-renders it. State is preserved.

### 4. Children without keys: positional matching

React compares children index by index. Appending to the end is cheap, but inserting at the front makes React treat every item as changed.

### 5. Children with keys: identity matching

React uses keys to match children in the old tree with children in the new tree. If key `3` is new and keys `1` and `2` already exist, React knows `3` is new and `1` and `2` just moved. It reorders DOM nodes instead of rebuilding them.

```jsx
// Good: stable, unique ID
{todos.map(t => <TodoItem key={t.id} todo={t} />)}

// Bad: index as key breaks on reorder, insert or delete
{todos.map((t, i) => <TodoItem key={i} todo={t} />)}
```

Using the index as a key causes element reordering bugs and reconciliation mismatches. For example, inputs may show the wrong values after a delete.

## Fiber: How Reconciliation Runs

Before React 16, reconciliation was synchronous and recursive. A big update could block the main thread and freeze the UI. **Fiber** is the rewrite that fixed this.

- Rendering work is split into small units called fibers. Each fiber represents a component and its properties.
- React builds a work-in-progress fiber tree while keeping the current committed tree intact.
- React reuses an existing fiber when type and key match. It marks fibers with effect flags (placement, update, deletion), which are applied at commit.
- With concurrent rendering, React can pause reconciliation when something more urgent comes up, like a user typing, and resume later.

| Phase | What happens | Interruptible? |
|---|---|---|
| **Render / reconcile** | Build the new fiber tree, diff, mark effects | Yes (concurrent mode) |
| **Commit** | Apply DOM mutations, run layout effects | No (must be atomic) |

## Practical Takeaways

- Use stable, unique keys from data IDs, never array indexes for dynamic lists.
- Don't define components inside other components. A new function identity each render means a different type, so React remounts the whole subtree.
- Keep component types consistent in the same position to preserve state.
- Use `React.memo`, `useMemo` and `useCallback` to skip reconciling unchanged subtrees.
- Use `startTransition` for non-urgent updates so Fiber can prioritize urgent ones.

## Video Resources

I could not verify a direct YouTube URL, so here are two reliable routes:

- **Video attached to a written guide**: the HackerNoon article below has an animated YouTube video on the Virtual DOM at the end of the post.
  https://hackernoon.com/virtual-dom-reconciliation-and-diffing-algorithm-explained-simply-ycn34gr
- **YouTube searches that work well**:
  - `React reconciliation diffing algorithm explained`
  - `React Fiber architecture explained`

