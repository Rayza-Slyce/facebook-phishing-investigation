# Investigation: Facebook “Who’s Stalking Your Profile?” Scam

---

## Initial Observation

While using Facebook, I was tagged in a post from a friend’s account that immediately looked suspicious. The post was promoting a feature that claims to show who has been viewing your profile, with the text:

> “Who’s Stalking Your Profile? See names in 30 seconds…”

The post included a link to an external website (`bildnews33.com`) and had tagged a large number of people (over 90 users). This tagging behaviour appeared to be used to draw attention and encourage people to click the link.

The post also included a comment from the same account saying:

> “It’s really funny how many familiar faces are checking out my profile… See for yourselves who’s been visiting!”

This wording did not seem natural and appeared to be a generic message designed to increase engagement.

To check visibility, I viewed the profile from a separate burner Facebook account that is not connected to the user. The post was not visible there, but it was still visible on my personal account. This suggests the post may have been set to “friends only”, meaning it is specifically targeting people who are more likely to trust the source.

Based on these signs, I suspected that:
- The account may have been compromised  
- The link may be part of a phishing or scam campaign  
- The tagging behaviour was being used to spread the link quickly  

A screenshot of the post was taken as initial evidence.

![Initial Facebook Post](01_initial_facebook_post.png)

---

## Evidence Collected

The following evidence was collected at the initial stage:

- Screenshot of the Facebook post  
- Full Facebook redirect link (copied safely using “Copy Link”)  
- Visible domain name (`bildnews33.com`)  
- Observation of different visibility between accounts  

The link was not clicked directly from the main browser. Instead, it was analysed in a controlled environment.

---

## Link Extraction & Decoding

The copied link used a Facebook redirect wrapper (`l.facebook.com`). After decoding the `u=` parameter, the real destination was identified as:

`https://bildnews33.com/p2-en/?utm_campaign=1220931354&tsid=8&fqty=1500&date_val=25_mar&pnum=090e6fff671c8fc7&com_id=28`

The URL contained multiple tracking parameters such as:
- `utm_campaign`
- `tsid`
- `pnum`
- `com_id`

This suggested the link was part of a tracked campaign rather than a simple static page.

---

## Passive Recon (Domain Analysis)

Initial checks were carried out using `whois`, `dig`, and `curl`.

Findings:
- The domain is relatively new  
- Privacy protection is enabled  
- Cloudflare is being used as a proxy  

Direct access to the root domain did not immediately show malicious content, suggesting possible conditional behaviour.

![WHOIS Summary](02_whois_summary.png)

---

## Behaviour Analysis (Redirect Chain and Landing Page)

The link was opened in Burp Suite’s browser to observe behaviour safely.

### Redirect Chain

The following sequence was observed:

1. `l.facebook.com`  
2. `bildnews33.com`  
3. `brucialseffset.com`  
4. `bildnachricht.com`  

![Redirect Chain - Part 1](03_redirect_chain_1.png)  
![Redirect Chain - Part 2](03_redirect_chain_2.png)

---

### Landing Page

The final page displayed:

> “(33) people have viewed your profile in the last 2 days!”

With a button labelled:

> “Show List”

![Landing Page](04_landing_page.png)

Tracking requests such as `/trk/ping` were observed, indicating that user activity is being monitored.

---

## Interaction Analysis (Post-Click Behaviour)

After clicking “Show List”, the page remained on the same domain:

- `bildnachricht.com`

A login prompt was displayed:

> “Enter login to view the full list.”

The page included:
- Mobile number or email field  
- Continue button  

![Login Page](06_post_click_page.png)

---

### Network Activity Observed

Two key POST requests were captured:

- `/trk/click` → tracks user interaction  
- `/api/init-login` → initialises login session  

This indicates that the site is preparing to collect user credentials.

---

## Domain & Infrastructure Analysis

Three domains were involved in the attack chain:

- `bildnews33.com` → entry point  
- `brucialseffset.com` → redirect layer  
- `bildnachricht.com` → credential harvesting page  

Observations:

- All domains are recently registered  
- Privacy protection is used  
- Different providers are used:
  - Cloudflare  
  - AWS CloudFront  

This suggests a layered setup designed to:
- obscure the attack chain  
- track users  
- deliver the final payload  

---

## Overall Assessment

This is a multi-stage credential harvesting campaign.

The attack flow is:

1. Social engineering via Facebook post  
2. Redirect chain across multiple domains  
3. Engagement page to build curiosity  
4. Fake login page to collect credentials  

---

## Indicators of Compromise (IOCs)

### Domains
- bildnews33.com  
- brucialseffset.com  
- bildnachricht.com  

### Endpoints
- /trk/ping  
- /trk/click  
- /api/init-login  

---

## Recommendations

- Avoid clicking suspicious links  
- Do not enter credentials on external sites  
- Enable two-factor authentication (2FA)  
- Review account activity  
- Remove unknown apps  

---

## Ethical Considerations

- No real credentials were entered  
- Analysis was conducted in a controlled environment  
- No actions were taken that could impact others  

---

## Responsible Disclosure

This phishing campaign was reported to the relevant platforms and service providers.

### Reports Submitted

- **Facebook**  
  - The original post was reported via Facebook’s reporting system  
  - Result: Post removed for violating Community Standards  

- **Cloudflare**  
  - Domains reported as phishing infrastructure  
  - Result: Phishing warning page deployed on `bildnachricht.com`  

- **Amazon Web Services (AWS)**  
  - Report submitted regarding hosting infrastructure used in the redirect chain  

- **Namecheap (Registrar)**  
  - Report submitted for `bildnachricht.com`  

- **Key-Systems (Registrar)**  
  - Report submitted for `brucialseffset.com`  

### Outcome

- The original Facebook post was removed
- At least one domain (`bildnachricht.com`) is now actively flagged as phishing
- Hosting and domain providers acknowledged the reports and are investigating

### Notes

- No sensitive data was submitted during testing  
- All analysis was conducted in a controlled lab environment (VM)  
- No interaction was made with real user accounts or credentials  

---

## Post-Takedown Activity (Infrastructure Evolution)

Following reporting and partial takedown of the phishing infrastructure, changes in behaviour were observed.

### Cloudflare Intervention

The domain `bildnachricht.com` is now flagged with a phishing warning:

![Cloudflare Warning](07_cloudflare_warning.png)

This indicates that the domain has been reported and classified as malicious.

---

### Infrastructure Adaptation

Despite this, the campaign remains active and has adapted:

- `bildnews33.com` → still active entry point  
- `brucialseffset.com` → now returns HTTP 404 when accessed directly  
- New domain introduced: `profilestalkers.com`  

### Updated Redirect Chain

1. `bildnews33.com`  
2. `brucialseffset.com`  
3. `profilestalkers.com` (new final destination)

---

### New Landing Page

The phishing content has been moved to a new domain:

![New Landing Page](08_new_landing_page.png)

The page continues to use the same social engineering tactic:
- Claims users can see who viewed their profile  
- Encourages clicking through to reveal a “visitor list”  
- Likely leads to credential harvesting  

---

### Observations

- The threat actors are actively maintaining the campaign  
- Infrastructure is rotated after detection or blocking  
- Use of multiple domains suggests a scalable phishing setup  
- 404 responses may indicate partial takedown or defensive evasion  

---

### Assessment

This behaviour is consistent with:

- Commodity phishing kits  
- Affiliate-style traffic funnels  
- Rapid domain rotation to evade detection  

---

### Next Steps

Further investigation planned into:

- `profilestalkers.com` hosting and ownership  
- Additional domains linked to the campaign  
- Potential reuse of infrastructure across other phishing campaigns  

---
