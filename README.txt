● To turn off the Tag # requirement in the future, you'll need to modify js/app.js in two locations:

  Location 1: Main Table Count Submission (around line 235-260)

  Find this validation block in the submit count button event listener:

  // --- Validation Logic ---
  let isValid = true;
  hideNotification();
  tagInput.classList.remove("input-error");  // ← REMOVE THIS LINE
  quantityInput.classList.remove("input-error");
  uomSelect.classList.remove("input-error");
  locationSelect.classList.remove("input-error");

  if (!tagInput.value) {  // ← REMOVE THIS ENTIRE IF BLOCK
    tagInput.classList.add("input-error");
    isValid = false;
  }
  if (!locationSelect.value) {
    locationSelect.classList.add("input-error");
    isValid = false;
  }
  // ... rest of validation

  And update the error message from:
  showNotification("Please fill out all required fields (Tag #, Location, UOM, and QTY).");
  To:
  showNotification("Please fill out all required fields (Location, UOM, and QTY).");

  Location 2: "Add New Item" Modal (around line 427-433)

  Find the requiredFields array and remove "tagNo":

  const requiredFields = [
    "tagNo",        // ← REMOVE THIS LINE
    "description",
    "modal-location",
    "modal-uom",
    "quantity",
  ];

  That's It!

  After making these two changes in js/app.js, Tag # will be optional instead of required. The field will still appear in the interface and store data if entered, but users won't be
  blocked from submitting without it.