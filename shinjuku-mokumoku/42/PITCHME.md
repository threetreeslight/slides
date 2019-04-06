### Page Object Pattern

@threetreeslight

on shinjuku mokumoku programming #42

---

## Who

![](https://avatars3.githubusercontent.com/u/1057490?s=200&v=4)

VP of Engineering at [Repro](https://repro.io)

実態は人事オジサン😇

---

## 今日やること

ShinjukuMokumoku Eventの作成を自動化できたので、Page Object PatternにRefactoringする

---

# What's
# Page Object?

---

## E2E is fragile

HTML Elementを直接指定したE2E testのぶっ壊れやすさよ

しかもUIは非同期で並行処理することができる

そんなUIに変更が入ったらテストの修正めんどい😇

---

## So, We'll Wrap element

- A page object wraps an HTML page, or fragment, with an application-specific API,
- Concurrency issues are another topic that a page object can encapsulate.

---?image=https://martinfowler.com/bliki/images/pageObject/pageObject.png&size=auto 100%

---

## Rule

- the significant elements on a page
- page object operations should return fundamental types or other page objects
- no assertions in page objects

---

## Tips

- rule of thumb
  - 親指でざっくり測ることから、経験則などのイデオムとして利用されるらしい

---

## Appendix

- [martin fowler - PageObject](https://martinfowler.com/bliki/PageObject.html)
- [SeleniumHQ - Page Objects](https://github.com/SeleniumHQ/selenium/wiki/PageObjects)
- [Seleniumデザインパターン & ベストプラクティス](https://www.oreilly.co.jp/books/9784873117423/)

