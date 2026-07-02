Circular Array Loop (LC 457)

Pattern: Two Pointers — Floyd's Cycle Detection (Slow & Fast)
Difficulty: Medium
LeetCode: 457. Circular Array Loop


Problem

Diya gaya hai ek circular array nums[] jisme har element batata hai ki current index se kitna aur kis direction (positive = aage, negative = peeche) move karna hai. Check karna hai ki array me koi valid cycle exist karta hai ya nahi.

Valid cycle ki conditions:


Cycle ki length > 1 honi chahiye (single element apne aap par loop nahi maanा jaata).
Poore cycle ke saare elements ka direction same hona chahiye (sab positive, ya sab negative — beech me direction switch nahi ho sakta).



Approach

Brute Force

Har index i se traversal start karo, ek visited set/array maintain karo us particular traversal ke liye, jab tak repeat index na mile ya direction change na ho jaye. Har baar fresh visited set banate ho.


Time: O(n²) worst case (har start se poora chain traverse)
Space: O(n) visited array ke liye


Better

Global visited array rakho taaki ek baar explore kiya hua index dobara start point na bane. Isse redundant traversals kam ho jaate hain, but abhi bhi cycle detect karne ke liye extra visited set chahiye har traversal me.


Time: O(n) amortized, lekin implementation thoda clunky
Space: O(n)


Optimal — Floyd's Cycle Detection (Slow & Fast Pointers)

Linked list wale "Happy Number" / "Cycle Detection" jaisa hi idea — bas yaha "next node" ka matlab hai (i + nums[i]) % n.

Har unvisited index i se:


slow ek step move karo, fast do steps move karo (dono same direction check ke saath).
Agar kabhi direction change ho ya single-element self-loop mile → is starting point se cycle possible nahi, break kar do.
Agar slow == fast mile (aur cycle length > 1) → valid cycle mil gaya, return true.
Agar is i se koi cycle nahi mila, to poore path ke indices ko 0 maar do (visited mark) taaki dobara explore na ho — isse overall traversal O(n) rehta hai.



Time: O(n) — har index zyada se zyada ek baar hi properly traverse hota hai
Space: O(1) — extra visited array ki zaroorat nahi, nums[] ko hi in-place mark kar rahe hain



Dry Run (example)

nums = [2, -1, 1, 2, 2]

i = 0: nums[0] = 2 (forward)
  slow: 0 -> 2         (0 + 2 = 2)
  fast: 0 -> 2 -> 4     (2 + 1 = 3? wait, nums[2]=1 so 2->3, nums[3]=2 so 3->0... )

  Actual trace:
  slow = next(0) = (0+2)%5 = 2
  fast = next(next(0)) = next(2) = (2+1)%5 = 3, then next(3) = (3+2)%5 = 0
  slow(2) != fast(0) -> continue

  slow = next(2) = 3
  fast = next(next(0's new fast=0)) -> fast = next(0)=2, next(2)=3
  slow(3) == fast(3) -> check length > 1 -> YES -> return true

Cycle: 0 -> 2 -> 3 -> 0 (length 3, sab positive) ✅ valid cycle mila.


Complexity (Optimal)

TimeSpaceOptimal (Floyd's + marking)O(n)O(1)


Key Insights / Gotchas


Circular index nikalne ke liye negative mod ka dhyan rakho: ((i + nums[i]) % n + n) % n, warna Java me negative remainder aa sakta hai.
Direction check har step pe karna zaroori hai — sirf starting element ka sign dekh ke assume mat karo poora cycle same direction hoga.
Single element self-loop (nextIdx == idx) ko invalid maano — cycle length > 1 chahiye.
Visited path ko 0 se mark karna hi O(n) banata hai, warna worst case O(n²) reh jaata hai.
Yeh bilkul Linked List Cycle Detection (Floyd's) jaisa hi mental model hai — bas "pointer.next" ki jagah "array index formula" use ho raha hai.
