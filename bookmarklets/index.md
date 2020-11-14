# Bookmarklets

Toggle strike through on `click`. Markdown does not support links with javascript so to "install" it you'll have to add a bookmark, joint the code below into one line and paste into the bookmark.

```js
javascript: (() => {
  document.head.insertAdjacentHTML('beforeend', '<style>.struck {text-decoration: line-through}</style>');
  document.addEventListener('click', e => e.target.classList.toggle('struck'));
})()
```

