# 修复GameRoom.handlePlay()中列表排序问题

> **Workspace:** /home/jiangyong/ws/bytedance/solo0601/JY-062/JY-062-21

---

**User**

GameRoom.handlePlay()将玩家手牌List直接传入CardLogic.getPlayType()，该方法内部对cards进行Collections.sort()修改了原始列表顺序，需在传入前做防御性拷贝，避免影响后续渲染和比较。

**Assistant**

### Relevant Code Snippets
1. /home/jiangyong/ws/bytedance/solo0601/JY-062/JY-062-21/src/main/java/com/ddz/server/GameRoom.java:L1-L100
  — This file contains the GameRoom class which likely implements handlePlay method for card gameplay logic.
2. /home/jiangyong/ws/bytedance/solo0601/JY-062/JY-062-21/src/main/java/com/ddz/server/CardLogic.java:L1-L100
  — This file contains the CardLogic class which likely implements getPlayType method for determining card play types.
3. /home/jiangyong/ws/bytedance/solo0601/JY-062/JY-062-21/src/main/java/com/ddz/server/CardLogic.java:L100-L200
  — This file contains the CardLogic class with possible defensive copy and sort operations in card handling methods.
4. /home/jiangyong/ws/bytedance/solo0601/JY-062/JY-062-21/src/main/java/com/ddz/server/GameRoom.java:L100-L200
  — This file contains GameRoom class implementation with potential defensive copy and sort operations in handling player actions.

我现在清楚地看到问题了。在 [CardLogic.java](file:///home/jiangyong/ws/bytedance/solo0601/JY-062/JY-062-21/src/main/java/com/ddz/server/CardLogic.java#L45-L49) 中，`getPlayType()` 直接对传入的 `cards` 列表调用 `cards.sort()`，这会修改原始列表。在 [GameRoom.java](file:///home/jiangyong/ws/bytedance/solo0601/JY-062/JY-062-21/src/main/java/com/ddz/server/GameRoom.java#L216) 中，`handlePlay()` 直接将 `cards` 参数传入 `getPlayType()`，并且相同的 `cards` 列表随后用于渲染和比较（例如第 246 行的 `lastPlayedCards = cards`，第 252 行的 `formatCardsForPrompt(cards)`）。

修复方法是在 `handlePlay()` 中将 `cards` 传入 `getPlayType()` 之前，创建一个防御性副本。

修复完成。问题分析如下：

**问题根因**：[CardLogic.getPlayType()](file:///home/jiangyong/ws/bytedance/solo0601/JY-062/JY-062-21/src/main/java/com/ddz/server/CardLogic.java#L49) 在第 49 行直接对传入的 `cards` 列表调用了 `cards.sort(Comparator.comparingInt(Card::getValue))`，这会就地修改原始列表的顺序。而 [GameRoom.handlePlay()](file:///home/jiangyong/ws/bytedance/solo0601/JY-062/JY-062-21/src/main/java/com/ddz/server/GameRoom.java#L216) 将 `cards` 直接传入该方法，导致原始手牌列表被排序，进而影响后续的渲染（`formatCardsForPrompt`）和比较逻辑（`lastPlayedCards = cards` 赋值）。

**修复方式**：在第 216 行调用 `getPlayType()` 时，传入 `new ArrayList<>(cards)` 作为防御性拷贝，确保 `getPlayType()` 内部的 `sort()` 操作只影响副本，不会篡改原始列表的顺序。

---

