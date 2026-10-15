# Shelf Scan

A phone web app that turns "take a photo of every empty shelf spot and send it to the manager" into a vendor-sorted restock list.

I work in the non-food section of a supermarket. Each product belongs to a vendor (identified by a store vendor number), and restocking means figuring out which vendor every missing item comes from. Shelf Scan lets you point a phone at a barcode: it recognizes the product, shows its vendor instantly, and builds a restock list grouped by vendor that can be exported as a spreadsheet.

> This public version uses made-up vendors and household products only. No real store data is included.

**Live demo:** `https://<your-username>.github.io/shelf-scan/`

## Features

- **Live barcode scanning** with the phone's rear camera. Line the barcode up inside the on-screen box: it turns amber when a code is detected and green when the read is confirmed, then the app scans automatically. No shutter button.
- **Misread protection.** A code is accepted only after two identical consecutive reads *and* a valid GTIN check digit.
- **Learn once, recognize forever.** The first time a barcode is seen, you assign a product name and vendor. Every later scan shows the vendor immediately.
- **Restock list grouped by vendor**, with per-vendor totals and a "copy this vendor's order" button.
- **CSV export** for Google Sheets or Excel (UTF-8 with BOM so Japanese text opens correctly; barcodes kept as text, not scientific notation).
- **Backup and restore** as JSON.
- **English / Japanese** interface toggle.
- Can be added to the iPhone home screen so it opens like an app.

## How it works

| Piece | Approach |
|---|---|
| Product lookup | Products are stored in a hash map keyed by barcode, so each scan is an O(1) lookup. |
| Vendor grouping | Items with a quantity above zero are bucketed by vendor number, then sorted by name within each vendor. |
| Check digit | GTIN rule: from the right (excluding the check digit), multiply digits by 3, 1, 3, 1…, sum, and the check digit makes the total a multiple of 10. Supports EAN-8, UPC-A, EAN-13/JAN and GTIN-14. |
| Scanning loop | Every ~110 ms the app crops only the region under the on-screen box from the video frame (accounting for `object-fit: cover` scaling), and decodes it with ZXing. Cropping makes decoding faster and stops it from reading barcodes outside the box. |
| Confirmation | A small state machine: `idle → seen → locked`. One read moves to `seen` (amber); a second identical read within 1.2 s moves to `locked` (green), plays a short beep, and pauses scanning. After saving, the same code is ignored for 2.5 s so it isn't scanned twice. |
| Storage | `localStorage` on the device. Nothing is sent to a server. |

## Tech

- HTML, CSS, vanilla JavaScript (single file, no build step)
- [ZXing](https://github.com/zxing-js/library) for barcode decoding
- `getUserMedia` for the camera, Web Audio for the confirmation beep
- Hosted on GitHub Pages (the camera requires HTTPS)

## Privacy

- Data stays in the browser on the device that scanned it.
- The app stores only barcode, product name, vendor number/name and a quantity. No prices, sales figures or personal information.

## Roadmap

- [ ] Java (Spring Boot) REST API with a database, so the vendor mapping can be shared between devices
- [ ] Date-stamped restock lists (one per shift)
- [ ] Unit tests for the check-digit and grouping logic (JUnit on the Java side)
- [ ] Import an existing barcode → vendor list from CSV

## Run locally

Open `index.html` in a desktop browser for everything except the camera. For the camera on a phone, use the GitHub Pages URL (HTTPS is required).
