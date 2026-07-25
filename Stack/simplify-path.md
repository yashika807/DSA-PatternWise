# Simplify Path — LeetCode 71

> 🔗 **[Interactive Visualizer dekho »](https://yashika807.github.io/DSA-PatternWise/Stack/simplify-path.html)**
> _(folder-by-folder push/pop, live path breadcrumb, synced Java code)_

**Difficulty:** Medium · **Pattern:** Stack (path segments) · **Tags:** String, Stack

---

## Problem

Ek Unix-style absolute file path `path` diya hai. Use **simplified canonical path** mein convert karo.

Rules:
- `.` matlab current directory — ignore karo.
- `..` matlab parent directory — ek level **upar** jao.
- Multiple consecutive slashes (`//`) ek single `/` jaisa treat hote hain.
- Canonical path hamesha `/` se start hota hai, aur folders ke beech single `/`, koi trailing `/` nahi (root ke alawa).

```
Input:  path = "/home/user/Documents/../Pictures"
Output: "/home/user/Pictures"
```

---

## The story (yaad rakhne ke liye) 🧠

Socho tum ek **file explorer** mein folders ke andar ghus rahe ho — jaise ek **undo history** ya **breadcrumb trail**. Har baar jab tum ek folder mein jaate ho, uska naam breadcrumb mein **push** ho jaata hai. Jab `..` milta hai (matlab "**back**" button dabaya), tum breadcrumb ka **aakhri folder pop** kar dete ho — wapas ek level upar.

- `path` ko `/` se **split** karo — har piece ek "instruction" hai.
- Piece **empty** hai (multiple slashes ki wajah se) ya `"."` hai → **ignore karo**, kuch mat karo.
- Piece `".."` hai → stack **pop** karo (agar stack empty hai to bhi kuch mat karo — root se aur upar nahi ja sakte).
- Koi bhi **real folder name** (`"home"`, `"Documents"`, etc.) → **push karo**.

Aakhir mein stack ke andar (bottom se top tak) jo folders bache hain, unhe `/` se jodo — wahi canonical path hai.

> 💡 Yeh bhi ek **stack-of-segments** pattern hai — bilkul `balanced-parentheses` jaisa "push on open, pop on close" mental model, bas yahan "open" = naya folder name, "close" = `..`.

---

## Approach 1 — Brute force 🟥

Path ko manually character-by-character parse karo, har segment ko extract karo, aur ek `ArrayList` ko manually manage karo (koi built-in stack use na karke).

```java
class Solution {
    public String simplifyPath(String path) {
        List<String> folders = new ArrayList<>();
        int i = 0, n = path.length();

        while (i < n) {
            while (i < n && path.charAt(i) == '/') i++;      // skip slashes
            int start = i;
            while (i < n && path.charAt(i) != '/') i++;       // read segment
            String seg = path.substring(start, i);

            if (seg.isEmpty() || seg.equals(".")) continue;
            if (seg.equals("..")) {
                if (!folders.isEmpty()) folders.remove(folders.size() - 1);
            } else {
                folders.add(seg);
            }
        }

        return "/" + String.join("/", folders);
    }
}
```

**Yeh technically stack ka hi use kar raha hai (`ArrayList` ko end se add/remove)** — par manual character parsing zaroori nahi hai jab Java ka `split("/")` yehi kaam clean tareeke se kar deta hai. Isse thoda verbose aur error-prone (off-by-one bugs) hota hai.

---

## Approach 2 — Optimal (Stack) ✅

`split("/")` se segments nikaalo, phir stack se process karo.

```java
class Solution {
    public String simplifyPath(String path) {
        Deque<String> stack = new ArrayDeque<>();
        String[] parts = path.split("/");     // multiple slashes → empty strings automatically

        for (String part : parts) {
            if (part.isEmpty() || part.equals(".")) {
                continue;                       // no-op segment
            } else if (part.equals("..")) {
                if (!stack.isEmpty()) stack.pop();   // go up one level (if possible)
            } else {
                stack.push(part);               // real folder name
            }
        }

        // stack top = last folder; build path bottom → top
        StringBuilder sb = new StringBuilder();
        Iterator<String> it = stack.descendingIterator();   // bottom → top order
        while (it.hasNext()) {
            sb.append("/").append(it.next());
        }

        return sb.length() == 0 ? "/" : sb.toString();
    }
}
```

**Kyun kaam karta hai:** `split("/")` khud multiple consecutive slashes ko empty-string segments mein tod deta hai, jo humara `.isEmpty()` check automatically skip kar deta hai — separate "collapse slashes" logic likhne ki zaroorat nahi. Stack sirf `push`/`pop` ke through directory navigation simulate karta hai.

---

## Dry run — `"/home/user/Documents/../Pictures"`

`split("/")` → `["", "home", "user", "Documents", "..", "Pictures"]`

| part | Type | Action | Stack (bottom→top) |
|:---:|:---|:---|:---|
| `""` | empty (leading slash) | skip | `[]` |
| `home` | folder | push | `[home]` |
| `user` | folder | push | `[home, user]` |
| `Documents` | folder | push | `[home, user, Documents]` |
| `..` | go up | pop | `[home, user]` |
| `Pictures` | folder | push | `[home, user, Pictures]` |

**Final stack (bottom→top):** `[home, user, Pictures]` → **`"/home/user/Pictures"`** ✅

---

## Complexity

| Approach | Time | Space | Notes |
|---|:---:|:---:|---|
| Manual char parsing | `O(n)` | `O(n)` | more index-management, easier to bug |
| Stack + `split("/")` | `O(n)` | `O(n)` | clean, idiomatic |

---

## Gotchas 🪤

- **`..` on empty stack = no-op, na ki crash** — root se upar jaane ki koshish (`"/../"`) simply ignore ho jaati hai, `-1` index ya exception nahi aani chahiye.
- **`split("/")` leading slash se ek empty string deta hai** (`"/home"` → `["", "home"]`) — isliye empty-string check zaroori hai, warna galat segment process ho jaayega.
- **Trailing slash bhi automatically handle ho jaata hai** — `"/home/"` → `["", "home"]` (Java ka `split` trailing empty strings drop kar deta hai by default), so extra handling ki zaroorat nahi.
- **`descendingIterator()` (ya reverse loop) zaroori hai** — `ArrayDeque` mein `push` head pe add karta hai, isliye seedha iterate karne se **reverse order** milega. Bottom-to-top order ke liye descending iterator use karo.
- **Empty result case** — agar sab kuch `..`/`.` ho ya path sirf `"/"` ho, stack empty reh jaata hai → return sirf `"/"` (root), empty string nahi.
- **Folder names mein `.` ya `..` jaisa dikhne wala kuch bhi ho sakta hai** (jaise `"..."` — teen dots) — yeh ek **valid folder name** hai, `".."` ke barabar nahi, isliye `.equals("..")` exact match use karo, `contains` ya `startsWith` nahi.
