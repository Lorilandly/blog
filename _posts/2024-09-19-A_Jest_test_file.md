---
title: "A Jest test file"
date: 2024-09-19
---

```javascript
describe('Demo Test Suite', () => {

  test('Demo test', () => {
        expect(1).toBe(1);
  });

  test('Demo bad test', () => {
        expect(8 + 8).toBe(0);
  });
});
```