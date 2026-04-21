# PES1UG25CS842 - PES Version Control System

## Lab Report

### Phase 5: Branching and Checkout Analysis

**Q5.1: How would you implement `pes checkout <branch>`?**

To implement checkout, three things must change in `.pes/`: HEAD must be updated to point to the new branch (`ref: refs/heads/<branch>`), the branch ref file must be read to get the target commit hash, and the working directory must be updated to match the target commit's tree. The complexity comes from updating the working directory — you must read the target commit's tree, compare it to the current tree, delete files that don't exist in the target, and write files that do. This requires recursively walking both trees and applying the differences.

**Q5.2: How to detect dirty working directory conflicts?**

For each file in the index, compute its current SHA-256 hash and compare it to the stored hash in the index. If they differ, the file has been modified but not staged — this is an unstaged change. Then compare the index entries against the target branch's tree entries. If a file differs between the current index and the target tree, and the working directory also differs from the index, there is a conflict and checkout must refuse.

**Q5.3: What happens in detached HEAD state?**

When HEAD contains a commit hash directly instead of a branch reference, new commits are made but no branch pointer is updated. The commits exist in the object store but become unreachable once HEAD moves. To recover, the user can run `pes log` to find the commit hash, create a new branch pointing to it, then checkout that branch.

---

### Phase 6: Garbage Collection Analysis

**Q6.1: Algorithm for finding unreachable objects?**

Start from all branch refs in `.pes/refs/heads/`. For each branch, walk the commit chain following parent pointers. For each commit, mark its tree as reachable, then recursively mark all blobs and subtrees in that tree. Use a hash set to track all reachable object hashes. After traversal, scan all files in `.pes/objects/` and delete any whose hash is not in the reachable set. For 100,000 commits and 50 branches, you would visit approximately 100,000 commits plus their associated trees and blobs — potentially 500,000 to 1,000,000 objects total.

**Q6.2: Race condition between GC and commit?**

A concurrent commit first writes blobs and trees to the object store, then writes the commit object, then updates the branch ref. GC could scan reachable objects between the blob write and the commit write — at that point, the new blobs exist but aren't reachable from any ref yet, so GC deletes them. When the commit finishes and tries to reference those blobs, they are gone. Git avoids this by using a grace period — objects newer than a threshold (default 2 weeks) are never deleted by GC, giving in-progress operations time to complete.

---

## Screenshots

[See screenshots folder or inline images for all required phase screenshots]

