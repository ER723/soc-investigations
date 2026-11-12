# Investigation 01: Phishing email posing as PayPal (German)

**Analyst:** Anggasukmana Adisastra
**Source:** practice sample from a LetsDefend training exercise (an old email, dated Aug 2022)
**Verdict:** Phishing, high confidence
**Recommended action:** Block, search for the same mail across mailboxes, check for clicks

## Summary

A German-language email, styled as PayPal, tells the recipient they have a "PayPal gift card 1000" waiting and must click to confirm delivery. The sender is not PayPal. The link does not go to PayPal either: it points to an HTML page stored in a Google Cloud Storage bucket. Several headers do not agree with each other, and the message has the marks of a bulk phishing run. I did not open the link or the landing page, so the attacker's exact goal (credential or card-data theft) is inferred, not confirmed.

## What I looked at

The raw message source (headers and HTML body) and the rendered email, in Mozilla Thunderbird inside the lab VM.

## Timeline (from the headers)

| Time | Event |
|---|---|
| Mon 15 Aug 2022, 10:35:01 EDT | `Date` header on the message |
| 10:35:02 EDT (07:35:02 PDT) | Accepted by Google's mail servers. SPF checked at that moment: **pass** (for the return-path domain, see below) |
| Earlier hop | The message first passed through a host named `efianalytics[.]com` (216[.]244[.]76[.]116) |

## Evidence

**1. The sender is not PayPal.**
The display name reads `★P.A.Y.P.A.L★`, but the address is `IHKH0MFEWW@kodehexa[.]net`. A second header, `X-EMMAIL`, shows `Marrybenh@kodehexa[.]net`. Dots and stars in the display name are an attempt to look like PayPal while avoiding a simple name match.

**2. The headers name four unrelated domains.**

| Where | Value |
|---|---|
| From address | `kodehexa[.]net` |
| Return-Path | `bounce@rjttznyzjjzydnillquh[.]designclub[.]uk[.]com` |
| Sending host (HELO) | `foresthillrestaurant[.]com` |
| Reverse name of sending IP | `capchrist[.]org` (IP 134[.]195[.]196[.]43) |

A legitimate PayPal message would come from a PayPal domain throughout.

**3. SPF passed, and that proves little here.**
`Received-SPF: pass` means the sending IP is allowed to send for `designclub[.]uk[.]com`, the **return-path** domain. That is a domain the attacker controls, so the pass says nothing about whether the mail is really from PayPal. DKIM and DMARC results were not visible in my screenshots, so I have not assessed them.

**4. Bulk-mail and template signs.**
- `To:` shows a placeholder-style address (`[an18]@itlgopk[.]uk`) while the mail was delivered to a different, real mailbox (a hidden-recipient mass mailing).
- The `Message-Id` is malformed (`<-@vevida[.]net>`).
- The headers carry bulk-mailing tool markers (`List-Unsubscribe`, a Marketo-style ID, an Amazon SES feedback ID), which would not be on a real PayPal notice sent this way.
- The content type is `multipart/report; report-type=delivery-status`, which is what a bounce message uses. This looks like an attempt to get past filters.

**5. The body.**
- Subject (German): "We called you, you did not answer", followed by a phone number. This is a pretext to create urgency.
- Greeting is generic ("Hallo !"). The text claims the recipient is "customer no. 12819202501 of AU Paypal Rewards" and has been "waiting since 09.08.2022 for your confirmation". The reward is a "PayPal-Guthabenkarte 1000" (a 1000 gift card).
- The German has missing special letters (for example "Besttigung" instead of "Bestätigung"), a sign of poor encoding or machine-assembled text.
- The word "Email:" in the HTML is written as HTML character codes (`&#69;&#109;&#97;&#105;&#108;`), a trick to dodge text-matching filters.
- The "PayPal" logo is an image reading "Paypal .com".

**6. The link.**
The reward link and the button go to:
`hxxps://storage[.]googleapis[.]com/hqyoqzatqthj/aemmfcylvxeo[.]html#<random string>`
`storage[.]googleapis[.]com` is a real Google service, which is exactly why attackers use it: the domain is trusted and rarely blocked. A real PayPal page would be on a PayPal domain. The bucket and file names are random strings, and the page is a plain HTML file, which fits a fake landing page.

## Indicators of compromise (defanged)

| Type | Value |
|---|---|
| Sender address | IHKH0MFEWW@kodehexa[.]net |
| Return-Path | bounce@rjttznyzjjzydnillquh[.]designclub[.]uk[.]com |
| Sending IP | 134[.]195[.]196[.]43 |
| Earlier hop | efianalytics[.]com / 216[.]244[.]76[.]116 |
| HELO host | foresthillrestaurant[.]com |
| URL | hxxps://storage[.]googleapis[.]com/hqyoqzatqthj/aemmfcylvxeo[.]html |
| Unsubscribe host | unsub-sj[.]mktomail[.]com |

## Verdict

**Phishing.** The sender is not PayPal, the headers do not agree with each other, the lure is a too-good-to-be-true prize with a manufactured deadline, and the link goes to an HTML file in a public storage bucket instead of a PayPal address.

## Recommended actions

1. **Block** the sender domain, the return-path domain, the sending IP and the URL at the mail gateway and web proxy.
2. **Search** all mailboxes for the same sender, subject and URL, and delete matching messages.
3. **Check for clicks:** look in proxy or DNS logs for any visit to the storage URL. For anyone who clicked, reset their passwords, and check for sign-ins they don't recognise.
4. **Report** the bucket to Google Cloud abuse, and the email to PayPal's phishing reporting address.
5. **Tell users** what this kind of message looks like (prize or gift-card lure, generic greeting, urgency, odd sender).

## What I did not do

- I did not open the link, so I have not seen the landing page.
- I did not run reputation checks on the IP, domains or URL, or a sandbox analysis. These would be the next steps to confirm the attacker's goal.
- DKIM and DMARC results were not available in the data I captured.
