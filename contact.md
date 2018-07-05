---
layout: default
permalink: /contact.html
title: Contact
---

# Contact Me
I appreciate that you'd like to touch base with me. Use this simple form and I'll get back to you as soon as I can.

<form method="POST" action="https://formspree.io/contact@dcalkins.com">
  <input type="email" name="email" placeholder="Email Address" style="width: 100%;margin-bottom: 0.5em;font-size: 14px;font-family:Noto Sans, Helvetica Neue, Helvetica, Arial, sans-serif;padding: 0.9em;">
  <br />
  <textarea name="message" placeholder="Message" style="min-width: 100%;max-width: 100%;min-height: 100px;margin-bottom: 0.5em;font-size: 14px;font-family:Noto Sans, Helvetica Neue, Helvetica, Arial, sans-serif;padding: 0.9em;"></textarea>
  <br />
  <input type="hidden" name="_subject" value="Contact Email - dcalkins.com" />
  <input type="hidden" name="_next" value="{{ "/thanks.html" | relative_url }}" />
  <button type="submit" style="font-size:14px;font-family:Noto Sans, Helvetica Neue, Helvetica, Arial, sans-serif; font-size: 1em; text-transform: uppercase; font-weight: 600; border: 2px solid #267CB9; color: #267CB9; background: transparent; line-height: 1em; padding: 0.6em 0.9em;cursor:pointer;">Send</button>
</form>