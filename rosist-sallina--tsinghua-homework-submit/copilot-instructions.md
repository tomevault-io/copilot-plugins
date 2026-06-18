## tsinghua-homework-submit

> Correctly handle Tsinghua Learn homework workflows: locate the assignment, download the prompt, read the instructions, submit the right file, and verify the result.


# Tsinghua Homework Submit Skill

Use this skill for the full Tsinghua Learn homework workflow, not just uploading a file.

Goals:

1. Find the correct course and assignment page.
2. Download the assignment prompt or attached files.
3. Read the assignment instructions and due date.
4. Submit the correct local file reliably.
5. Avoid cross-course mistakes like uploading the wrong homework to the wrong class.

## When to use

Use this skill when:

- The user is already logged into Tsinghua Learn in Chrome.
- The course page, assignment page, or submission page is already open in the real browser, or can be reached from an existing tab.
- You need the complete workflow: download prompt + read instructions + submit solution.

## Core rules

1. Confirm the course first, then the assignment title, then the local file path, and only then submit.
2. Never guess the answer file only from “most recently created PDF”.
3. If the page says “upload failed” or “unsupported file type”, do not trust it immediately; first determine whether:
   - it is a frontend-only error, or
   - the server actually rejected the file.
4. Before submitting, always do a triple check:
   - current course name
   - current assignment title
   - absolute path of the local file to submit

## Recommended workflow

### Step 1: Find the Tsinghua Learn tab

```bash
alma browser tabs
```

Look for tabs like:

- `https://learn.tsinghua.edu.cn/...`
- titles that are course names, such as `Operating Systems`

If you are not already on an assignment page, navigate from the course page.

### Step 2: Confirm what page you are on

```bash
alma browser read <tabId>
alma browser read-dom <tabId>
```

Verify all of the following:

- whether this is the course homepage, assignment list page, assignment detail page, or submission page
- current course name
- current assignment title
- due date
- whether the page shows things like assignment attachments, answer attachments, or a submit button

## Locating a specific assignment from the assignment list

Assignment list pages are usually like:

- `/f/wlxt/kczy/zy/student/beforePageList?...`

You can extract the table with:

```bash
alma browser eval <tabId> "(() => {
  const rows=[...document.querySelectorAll('table tr')]
    .map(tr => [...tr.querySelectorAll('td')].map(td => td.innerText.trim().replace(/\\n+/g,' | ')))
    .filter(r => r.length);
  return JSON.stringify(rows, null, 2);
})()"
```

If you want the detail link for each row:

```bash
alma browser eval <tabId> "(() => {
  const rows=[...document.querySelectorAll('table tr')].map(tr => [...tr.querySelectorAll('td')]);
  const items=rows.filter(tds => tds.length >= 6).map(tds => ({
    title: tds[1]?.innerText.trim(),
    href: tds[1]?.querySelector('a')?.href || ''
  }));
  return JSON.stringify(items, null, 2);
})()"
```

Then open the target assignment detail page.

### Step 3: Read the assignment details and instructions

Assignment detail pages are usually like:

- `/f/wlxt/kczy/zy/student/viewZy?...`

Read them with:

```bash
alma browser read <tabId>
alma browser read-dom <tabId>
```

If `alma browser read` does not extract the content cleanly, fall back to:

```bash
alma browser eval <tabId> "document.body.innerText"
```

Extract at least:

- course name
- assignment title
- assignment instructions shown on the page
- due date
- names of attached prompt files

### Step 4: Download the assignment prompt correctly

Do not guess download URLs with curl first.

Instead, first inspect the DOM on the assignment detail page and collect the attachment links:

```bash
alma browser eval <tabId> "(() => JSON.stringify(
  [...document.querySelectorAll('a')]
    .map(a => ({text:(a.innerText||'').trim(), href:a.href}))
    .filter(x => x.text || x.href.includes('downloadFile')),
  null, 2
))()"
```

Common cases:

- the attachment filename itself is a link
- there is a separate `Download` link nearby
- the real download URL may include an `_csrf` parameter

After you have the link, prefer opening it in the browser so the site can use the existing login session:

```bash
alma browser open "<attachment-url>"
```

If the site says the file cannot be previewed and will be downloaded automatically, continue and confirm.

Then find the newly downloaded local file:

```bash
find ~/Downloads -maxdepth 1 -type f -mmin -5 -print0 | xargs -0 ls -lt | head
```

### Step 5: Read the downloaded prompt file

Once the file is on disk, read the local file itself instead of guessing from the preview page.

Examples:

```bash
Read /Users/.../Homework4.md
Read /Users/.../HW4.txt
```

If it is a PDF, prefer checking whether there is also a markdown, doc, or txt version first. If not, use an appropriate PDF-reading path afterward.

## Submitting homework: two strategies

### Strategy A: Prefer the page’s native submission flow

First open the submission page:

- `/f/wlxt/kczy/zy/student/tijiao?...`

If the detail page has a submit button, use that first. Otherwise, inspect page scripts or functions such as `goBtnF()` to find the target submission URL.

Once on the submission page, confirm these important elements:

- form: `#form_sn1`
- file input: `#fileupload`
- submit button: often `#goBtn` or `input[value='提交']`
- submission text field: `#s_documention` or `name='zynr'`

Check the form with:

```bash
alma browser eval <tabId> "(() => JSON.stringify({
  action: document.querySelector('#form_sn1')?.action || '',
  fileInput: !!document.querySelector('#fileupload'),
  submitBtn: document.querySelector('#goBtn')?.outerHTML || document.querySelector(\"input[value='提交']\")?.outerHTML || ''
}, null, 2))()"
```

After uploading a file, confirm that it really entered `input.files`:

```bash
alma browser eval <tabId> "(() => {
  const el=document.querySelector('#fileupload');
  return JSON.stringify({
    len: el?.files?.length || 0,
    name: el?.files?.[0]?.name || '',
    size: el?.files?.[0]?.size || 0
  }, null, 2);
})()"
```

Only continue if `len = 1`.

You can usually submit by calling the page’s own function:

```bash
alma browser eval <tabId> "daijiao(); 'submitted';"
```

After submission, the page often returns to the assignment list.

### Strategy B: If the page upload UI is unreliable, submit with browser-context POST

If the file picker, file input, or frontend scripts keep breaking, directly send a multipart request using the current browser session and XSRF token.

First read the form parameters:

```bash
alma browser eval <tabId> "(() => {
  const form=document.querySelector('#form_sn1');
  return JSON.stringify({
    xszyid: form?.querySelector(\"input[name='xszyid']\")?.value || '',
    isDeleted: form?.querySelector(\"input[name='isDeleted']\")?.value || '0',
    zynr: form?.querySelector(\"textarea[name='zynr']\")?.value || ''
  }, null, 2);
})()"
```

Convert the local file to base64:

```bash
python3 - <<'PY' > /tmp/alma_hw_b64.txt
import base64
with open('/ABS/PATH/TO/FILE.pdf','rb') as f:
    print(base64.b64encode(f.read()).decode())
PY
```

Then construct `File` and `FormData` inside the page context:

```bash
b64=$(cat /tmp/alma_hw_b64.txt)
alma browser eval <tabId> "window.__almaSubmitResult='pending'; (function(){
  const b64='$b64';
  const bin=atob(b64);
  const arr=new Uint8Array(bin.length);
  for(let i=0;i<bin.length;i++) arr[i]=bin.charCodeAt(i);
  const fd=new FormData();
  fd.append('xszyid','<xszyid>');
  fd.append('isDeleted','0');
  fd.append('zynr','');
  fd.append('fileupload', new File([arr], '<filename.pdf>', {type:'application/pdf'}));
  fetch('/b/wlxt/kczy/zy/student/tjzy', {
    method:'POST',
    body:fd,
    headers:{'X-XSRF-TOKEN':(document.cookie.match(/(?:^|; )XSRF-TOKEN=([^;]+)/)||[])[1]||''},
    credentials:'include'
  }).then(async r => {
    window.__almaSubmitResult = JSON.stringify({
      ok:r.ok,
      status:r.status,
      text:(await r.text()).slice(0,1500)
    });
  }).catch(e => window.__almaSubmitResult='ERR:'+e);
  return 'started';
})()"
```

Poll the result:

```bash
sleep 3
alma browser eval <tabId> "window.__almaSubmitResult || 'missing'"
```

## How to verify that submission actually succeeded

Do not rely only on whether the page showed an error.

Always return to the assignment list page and inspect the target row to see whether it now shows an attachment size and submission status.

Example:

```bash
alma browser eval <tabId> "(() => {
  const rows=[...document.querySelectorAll('table tr')]
    .map(tr => [...tr.querySelectorAll('td')].map(td => td.innerText.trim().replace(/\\n+/g,' | ')))
    .filter(r => r.length);
  return JSON.stringify(rows, null, 2);
})()"
```

Typical success signals:

- the target assignment title is still present
- the row shows an attachment size such as `214.36K`
- the status column is no longer blank
- the row shows markers like `提交作业收藏备注` or other evidence of a recorded submission

## Most important anti-mistake checklist

Before every submission, confirm all of these:

1. Current course name is correct.
2. Current assignment title is correct.
3. Local absolute file path is correct.
4. The local filename belongs to this course, not some other course.
5. `input.files[0].name` matches the expected file.

A good final pre-submit summary is one short sentence like:

- Current page: Operating Systems / Homework 4
- File to submit: `/Users/.../OS/hw/Homework4.pdf`

## Common pitfalls

1. “Most recently created PDF” is not necessarily the homework for the current course.
   - Always inspect the absolute path and filename.
2. The page may keep showing an old “upload failed / unsupported file type” message.
   - That does not necessarily mean the current upload actually failed.
3. `ChromeRelayUpload` returning success does not guarantee the file entered the form.
   - Always inspect `input.files`.
4. It is easy to click the wrong button.
   - Some pages contain rich text editor toolbar buttons.
   - The real submit button is usually `#goBtn` or the bottom `input[value='提交']`.
5. Do not stop at the submission page.
   - Always go back to the assignment list and verify the recorded result.

## Recommended completion message

When the task is done, report in one short sentence:

- which course
- which assignment
- which file was submitted
- whether the assignment list now shows the attachment size

Example:

- `Re-submitted successfully: Operating Systems / Homework 4 now shows the correct 214.36K PDF in the assignment list.`

## Final takeaway

The correct order for Tsinghua Learn homework is not “upload first and check later”.

It should always be:

Find the right course and assignment → download and read the prompt → produce the correct file → verify the absolute path → upload → verify from the assignment list.

---
> Source: [Rosist-Sallina/tsinghua-homework-submit](https://github.com/Rosist-Sallina/tsinghua-homework-submit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
