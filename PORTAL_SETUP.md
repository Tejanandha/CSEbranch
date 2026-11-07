# CSE Student Portal — setup notes

The portal UI is in `portal.html` and is linked from the existing syllabus explorer menu.

## Current infrastructure

- Supabase project: `CSE Student Portal`
- Project ref: `uxaplounevvjdlieejff`
- Region: `ap-south-1`
- Project plan: Free (project creation quote was $0/month)
- Student data tables: profiles, student_allowlist, study_materials, assignments, submissions, announcements, daily_news, record_deadlines
- Private Storage bucket: `portal-files`
- Edge Function: `portal-ai`
- An initial portal admin address is pre-approved in the private Supabase allowlist; it is intentionally not repeated in this public repository.

## Important remaining provider setup

The code is deployed, but the external provider credentials/settings below are not available through the connected project-management tools. Do not paste secret keys into GitHub or into `portal.html`.

### 1. Authentication URL settings

In Supabase Dashboard → Authentication → URL Configuration:

- Site URL: `https://tejanandha.github.io/CSEbranch`
- Add redirect URL: `https://tejanandha.github.io/CSEbranch/portal.html`

### 2. Email magic links

The portal requests email magic links (not numeric OTP codes). The Supabase default email service is unsuitable for class login: it only sends to project-organization team addresses and currently allows 2 emails per hour. See the [Supabase SMTP guide](https://supabase.com/docs/guides/auth/auth-smtp).

A no-cost option to evaluate is [Brevo Free](https://www.brevo.com/), which currently includes SMTP/transactional email and 300 email sends per day. That is enough for an initial 80-student rollout, subject to Brevo approving the sender and account. See [Brevo's official free-plan limits](https://help.brevo.com/hc/en-us/articles/208580669-FAQs-What-are-the-limits-of-the-Free-plan).

1. Create a Brevo Free account and verify a sender email/domain.
2. In Brevo, create SMTP credentials (use the SMTP key, not your normal account password).
3. In Supabase Dashboard → Authentication → Emails → SMTP Settings, enable custom SMTP and enter the host, port, SMTP login, SMTP key, and verified sender.
4. In Supabase Dashboard → Authentication → Rate Limits, set the email send rate to 10 per hour if available. Supabase's custom SMTP default is 30 new users per hour.
5. Keep the Magic Link template link-based (use `{{ .ConfirmationURL }}`, not `{{ .Token }}`).

Do not put SMTP credentials in this repository or in the browser code.

### 3. Google sign-in

Enable Google in Supabase Dashboard → Authentication → Providers → Google. Add this callback URL to the Google OAuth client:

`https://uxaplounevvjdlieejff.supabase.co/auth/v1/callback`

Then enter the Google OAuth client ID and secret in Supabase. The portal already calls the Google OAuth flow.

### 4. AI assistant

The secure `portal-ai` Edge Function is deployed. To activate AI replies without exposing a key in the browser:

- Create a free-tier Gemini API key in Google AI Studio.
- Add it in Supabase Dashboard → Edge Functions → Secrets as `GEMINI_API_KEY`.

The function checks the user's Supabase session and approved profile before calling the AI service. Free-tier quota and availability are controlled by the provider.

### 5. Approve at least 80 students

Sign in as the initial admin, open Admin Studio, and import a CSV with this header:

```csv
email,name
student1@college.edu,Student One
student2@college.edu,Student Two
```

The allowlist can hold 80 or more students. Only allowlisted emails become approved; unlisted accounts remain pending. The admin panel also supports adding students one at a time.

## Persistent sign-in

Supabase Auth is configured in the client with session persistence and automatic token refresh. On the same browser profile/device, students remain signed in until they explicitly sign out, clear site data, or the session is revoked/invalidated. This is persistent device/browser sign-in, not a hardware-bound identity lock.

## Budget note

No paid Supabase plan, paid database add-on, or paid AI service is enabled by this implementation. Free-tier email/AI services have usage limits and can change. Do not upgrade or enable a paid provider without checking its current price first.
