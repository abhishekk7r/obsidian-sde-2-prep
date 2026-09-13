Pattern: DFS to claim one island + Multi-source BFS to bridge to the other
Two-phase approach — Phase 1: find & mark first island, Phase 2: expand through water until second island is hit
Phase 1 (DFS): find any cell with value 1, DFS to mark entire island as 2; while marking, push boundary cells (cells with at least one 0 neighbor) into a queue
Phase 2 (Multi-source BFS): all boundary cells start expanding simultaneously — process full level with size = q.size(), increment distance after each level, stop the moment any neighbor == 1 (that's island 2)
🔑 Expanding from one island is sufficient — you don't need to BFS from both islands; first contact = minimum bridge by BFS guarantee
🔑 Shape of the island doesn't matter — imagine dropping a stone at every boundary cell at the same time; ripples expand uniformly, whichever hits island 2 first wins regardless of T/L/irregular shape
⚠️ Mark water cells as visited (grid[nr][nc] = 2) AT PUSH TIME, not at pop time — without this, same cell gets pushed by multiple neighbors, duplicates break the level count guarantee; push and mark together, always
⚠️ Breaking out of nested loops to stop after finding first island — single break only exits inner loop; use a bool found flag and check it in outer loop
⚠️ Boundary cells with multiple water neighbors get pushed into queue multiple times during DFS — not a correctness bug (already marked 2) but causes redundant BFS work; minor inefficiency, not worth fixing in interview

Complexity: O(nm) time (DFS visits each cell once, BFS visits each cell once), O(nm) space (queue + recursion stack)