> **Moved.** This repository now lives at
> [`Hartwell-Labs/photoscrub`](https://github.com/Hartwell-Labs/photoscrub).
> GitHub redirects the old address, so old links still work — but a
> redirect is invisible. Anyone arriving from a bookmark never learns the
> project moved, or that it is maintained from the organisation rather than
> from a personal account.
>
> Copyright and history are unchanged. The repository was transferred, not
> copied, so every commit, release, tag and issue came with it.

# photoscrub

Shows you what your photos leak — then removes it.

Passive, local, no upload, no account. Windows and Linux.

## What it finds

| Finding | Level | Why it matters |
|---|---|---|
| **GPS coordinates** | HIGH | exact position, 6 decimal places |
| **Embedded thumbnail** | HIGH | a JPEG can carry a second, *complete* copy of the original image as a 160×120 preview. Stripping EXIF does not remove it |
| **Body serial number** | HIGH | identifies the device, and through purchase history, you |
| **XMP: City, Country, GPS** | HIGH | a second place for the same data — cleaning EXIF is not enough |
| **IPTC: byline, City, Country** | HIGH | editorial fields people forget about |
| **PNG tEXt / iTXt / zTXt / tIME / eXIf** | MED | author, description, comment, embedded EXIF |
| **Artist / Copyright / Software / Lens** | MED | who, with what, where |
| **Data appended after the image** | MED | invisible in preview, readable in a hex editor |
| **GPS in video (MP4/MOV udta atom)** | HIGH | video has no EXIF tags; location lives in the container |
| **Addresses, postcodes, phone numbers in free text** | MED | the leak nobody looks for, because it is not a tag |
| **ZIP: .DS_Store, Thumbs.db, ._sidecars** | LOW | usernames and full paths inside archives |

HEIC/HEIF (iPhone) and RAW extensions (DNG, CR2, NEF, ARW, ORF, RW2) are read.

## Rulesets

Every rule ships, and there is nothing to buy:

- **basic**: EXIF, GPS, serial, artist, and the thumbnail hidden in the JPEG.
- **full**: everything above plus HEIC, XMP, IPTC, PNG chunks, JPEG comments,
  data appended after the image, GPS in video, and PII in free text.

Rule sets are versioned so that a saved configuration stays meaningful: a profile
written against one version keeps applying against that version. That is a format
decision, not a paywall.

## The model

A small classifier we trained ourselves (`tools/train_model.py`) scores publish risk
from 15 features. It is **not** an LLM: it is logistic regression, 1.8 KB, trained on
8000 synthetic metadata profiles, 97.4% accuracy on held-out data. It runs offline and
every decision comes from coefficients you can read in `model.json`.

It is also how we found our own bug: it classified a PNG whose description contained
"Mieszkanie, ul. Rodła 12" as safe. That is why `has_free_text_pii` exists.

## Use

```
photoscrub PATH              # report only
photoscrub PATH --scrub OUT  # write metadata-free copies to OUT
photoscrub PATH --json       # machine readable
```

Originals are never modified. `--scrub` refuses to write into the source folder.

## Signing

Keys are still Ed25519-signed, which is what lets activation work with no network
and no phone home. That machinery is kept because it is how the offline builds
verify they were not tampered with — it no longer gates anything.

```
python3 tools/issue.py --email you@example.com --name "You" --ruleset 2
```

## License

MIT. See [`LICENSE`](LICENSE).

Free to use, modify and redistribute. Every ruleset ships in full — there is no
paid tier and no key to activate.
