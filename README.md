Happy to walk through it. Here's the mechanism end-to-end:

1. The trap: Google Forms expects its own HTML form

Normally when you fill out a Google Form and hit Submit, the page sends a real HTML form POST to a URL like:


https://docs.google.com/forms/d/e/<FORM_ID>/formResponse
with each answer keyed by a weird name like entry.1780254190 (that number is unique per question — Google generates it, it's not something you choose).

You found that entry ID by inspecting the form's DOM — the <input name="entry.1780254190" aria-label="Name"> you pasted is exactly that field. So we now know: "whatever value I send under the key entry.1780254190, Google will record as the answer to the Name question."

2. Skip Google's UI, send the same data yourself

Instead of a person visiting the actual Google Form, trip.html builds its own pretty little form (the heart emoji card), and when "Yes" is clicked, JS grabs whatever the person typed into the name box and stuffs it into a FormData object under that same key:


const formData = new FormData();
formData.append(NAME_ENTRY_ID, name);  // "entry.1780254190" -> "Nagavarshini"
Then it does a raw fetch(...) POST to formResponse, mimicking what Google's own form would have sent — see trip.html:439-446.

Google's server has no way to tell "did a human fill out my HTML form" from "did a POST request show up with the right field names" — it just records whatever arrives under entry.1780254190 as a new spreadsheet row.

3. Why mode: "no-cors"

Google Forms doesn't send CORS headers allowing cross-origin JS to read the response. Browsers block you from reading the response either way, so no-cors just tells fetch "fire the request, don't even try to give me the response back — I don't need it." That's why the code doesn't check success/failure; it can't. It just trusts the request went out and shows your success screen regardless (that's the "slightly hacky" part).

4. Why you had to dig through page source

There's no documented API for this — you're essentially reverse-engineering Google's own form submission by reading the HTML it renders. That's why you had to open dev tools, find the <input> for the Name question, and copy its name="entry.1780254190" attribute. Every question on a form gets its own random entry ID this way, so if you ever add more fields (like "excited level 1-10"), you'd repeat this same DOM-inspection step for each one.

The net effect: you get a free, no-backend way to collect form data — it lands directly in the Google Sheet behind your form, which you can already view/query/export — while the actual person-facing page is 100% your own custom design instead of Google's generic form UI.

Remote Control is active · Continue here, on your phone, or at claude.ai/code