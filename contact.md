---
layout: default
permalink: /contact.html
title: Contact
---

# Contact Me
I appreciate that you'd like to touch base with me. Use this simple form and I'll get back to you as soon as I can.

<form method="POST" action="https://formspree.io/contact@dcalkins.com">
  <input type="email" name="email" placeholder="Your email">
  <textarea name="message" placeholder="Your message"></textarea>
  <input type="hidden" name="_subject" value="Contact Email - dcalkins.com" />
  <input type="hidden" name="_next" value="{{ "/thanks.html" | relative_url }}" />
  <button type="submit">Send</button>
</form>