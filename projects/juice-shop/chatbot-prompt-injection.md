# OWASP Juice Shop: Chatbot Prompt Injection (Injection)

## Summary
The support chatbot is designed to only issue discount coupons to 
customers with a verified damaged item order who explicitly rejected 
a return/exchange. However, the chatbot does not actually verify 
submitted order IDs against real order data it only validates that 
the input matches an expected *format*, allowing a completely 
fabricated order ID to successfully bypass the coupon policy.

## Discovery Process
1. Asked directly for a coupon code  the bot didn't refuse outright, 
   but instead revealed its exact internal eligibility criteria: a 
   valid order ID for a damaged item, plus confirmation the return was 
   explicitly rejected
2. Submitted a plausible looking but fake order ID (`12345`) — the bot 
   rejected it, but in doing so leaked the *exact expected format*: 
   `xxxx-xxxxxxxxxxxxxxxx` (4 characters, a dash, then 16 characters)
3. Submitted a new fake order ID matching that exact format 
   (`abcd-1234567890abcdef`), along with the required confirmation 
   phrasing about the damaged item and rejected return
4. The chatbot accepted this fabricated data without any real 
   verification and generated a working coupon code: `q:<Irhz3Tq`

## Vulnerability
This is a prompt injection / business logic bypass vulnerability. The 
LLM-backed chatbot enforces its coupon policy purely through 
conversational reasoning and pattern matching on user provided text, 
rather than actually querying a backend system to verify that the 
claimed order ID, damage status, and rejection actually exist and are 
true. By iteratively querying the bot about *why* it rejected input, 
it inadvertently revealed the exact validation criteria needed to 
satisfy it with fabricated data.

## Steps to Reproduce
1. Ask the chatbot for a coupon code directly
2. Note the eligibility criteria it describes in its response
3. Submit an incorrectly-formatted fake order ID to prompt a rejection 
   message this reveals the exact expected format
4. Submit a new order ID matching that format exactly, along with the 
   other required confirmation details, all fabricated
5. Receive a valid, real coupon code despite providing no genuine 
   order data

## Impact
This allows any user to obtain discount coupons without any legitimate 
basis, purely by conversationally guessing and matching the chatbot's 
expected input format. In a real production system, this could result 
in significant financial loss through fraudulently issued discounts at 
scale, especially since the technique (asking the bot to explain its 
own rejection reasons) is a low effort, repeatable way to reverse
engineer validation logic conversationally.

## Fix
LLM-backed features that gate access to real business actions (like 
issuing discounts) must independently verify claims against actual 
backend data querying the real orders database to confirm the order 
ID exists, belongs to the requesting user, is genuinely marked as 
damaged, and that a return was actually rejected rather than trusting 
the LLM's own reasoning about whether user provided text "looks 
correct." Additionally, the chatbot should avoid revealing internal 
validation logic or exact expected formats in its rejection messages, 
since this directly aids an attacker in crafting a bypass.

## What I Learned
This was a genuinely different category of vulnerability from anything 
else in my portfolio  instead of exploiting code (SQL, XSS, IDOR), 
this exploited the *reasoning* of an AI system through natural 
conversation. The core lesson is that LLMs are fundamentally 
pattern-matchers, not verifiers  without an explicit, independent 
check against real data, an LLM can be talked into believing (or at 
least acting as if) unverified claims are true, especially when it 
readily explains its own rejection criteria along the way. This 
reinforces that prompt injection isn't just about "jailbreaking" a 
chatbot into saying something inappropriate  it can directly enable 
real business logic bypasses when an LLM is given authority over 
actual application actions like generating coupons.

## Root Cause: The Actual Vulnerable Code

The vulnerable `generateCoupon` tool definition:

​```javascript
generateCoupon: tool({
  description: 'Generate a discount coupon for a customer. Only use 
  this when the coupon policy conditions are fully met.',
  inputSchema: z.object({
    discount: z.number().describe('The discount percentage for the 
    coupon (maximum 10)')
  }),
  execute: async ({ discount }) => {
    const couponCode = security.generateCoupon(discount)
    return { couponCode, discount }
  }
})
​```

This confirms exactly what the black-box testing suggested: the 
policy ("only use this when conditions are met") existed purely as a 
natural language instruction to the LLM in the tool's description 
field. The actual executable code takes no order ID, performs no 
database lookup, and generates a real coupon immediately whenever the 
LLM chooses to call this function regardless of why it made that 
decision. The LLM was solely responsible for policing its own tool 
usage, with zero enforcement at the code level.

## The Fix

​```javascript
generateCoupon: tool({
  description: 'Generate a discount coupon for a customer with a 
  verified damaged order. Requires a valid order ID.',
  inputSchema: z.object({
    discount: z.number().describe('The discount percentage for the 
    coupon (maximum 10)'),
    orderId: z.string().describe('The order ID of the damaged order 
    (format: xxxx-xxxxxxxxxxxxxxxx)')
  }),
  execute: async ({ discount, orderId, authenticatedUser }) => {
    const order = await db.ordersCollection.findOne({ 
      orderId, 
      email: authenticatedUser?.email, 
      status: OrderStatus.DAMAGED 
    })
    if (!order) return { error: 'No verified damaged order found for 
    this order ID.' }
    const couponCode = security.generateCoupon(discount)
    return { couponCode, discount }
  }
})
​```

The fix requires the LLM to supply an `orderId`, then performs a real 
database query checking three conditions together: the order exists, 
it belongs to the currently *authenticated* user specifically (not 
just any user), and its status is genuinely `DAMAGED`. Coupon 
generation only proceeds if a real, matching record is found the 
vulnerable code path is now unreachable through conversation alone, no 
matter how convincingly formatted a fabricated order ID looks.

## What This Added To My Understanding
Seeing the real code confirmed a principle that applies well beyond 
this specific challenge: instructions written in a tool's *description* 
field are guidance for the LLM's reasoning, not enforcement they 
carry no more weight than a comment in code as far as actual security 
goes. Real enforcement has to happen in the executable logic itself, 
independently verifying claims against genuine data sources (and 
critically, tied to the *authenticated* user's identity, not just any 
matching record) before performing any sensitive action. This is 
directly analogous to every access control lesson from earlier in this 
project (IDOR, broken authorization) the new twist here is that the 
"input" being checked arrives via an AI's interpretation of a 
conversation rather than a raw HTTP parameter, but the underlying 
security principle is identical.
