# Releasing

The runbook for cutting a widget release and publishing it here. Source lives in a separate private repository; this repository carries only the built artifacts and documentation.

## 1. Build

**Build from a Windows-native terminal, not through WSL.** Invoke it from WSL like this:

```
cmd.exe /c "cd /d E:\Apps\MendixWidgets\importer-mendix-widgets\widgets\importer-review && npm run release"
```

WSL-native node running over the `/mnt/e` 9p filesystem takes roughly 20 minutes for the same build that finishes in about 70 seconds Windows-side.

**Use `npm run release`, never `npm run build`.** `npm run build` produces an unminified development bundle and must never be published. The release output lands in `dist/1.0.0/`.

## 2. Confirm the package contents

The `postrelease` hook injects `LICENSE`, `licensing.txt` and `NOTICE` into the `.mpk` root. Confirm they are there before publishing:

```
python3 -c "import zipfile,sys; print('\n'.join(zipfile.ZipFile(sys.argv[1]).namelist()))" dist/1.0.0/peralys.MassImporterReview.mpk
```

`LICENSE`, `licensing.txt`, `NOTICE`, `dependencies.txt` and `dependencies.json` must all be at the root, alongside `package.xml`.

## 3. Confirm no source maps

This is what keeps the source private. The release bundle must carry no `.map` entries and no source-map references:

```
python3 -c "
import zipfile,sys
z=zipfile.ZipFile(sys.argv[1])
print('map files:', [n for n in z.namelist() if n.endswith('.map')] or 'NONE')
for n in z.namelist():
    if n.endswith('.js') and '/' in n:
        d=z.read(n)
        print(n, 'sourceMappingURL' , b'sourceMappingURL' in d, 'sourcesContent', b'sourcesContent' in d)
" dist/1.0.0/peralys.MassImporterReview.mpk
```

Expect `NONE` and `False False`. Anything else means the build configuration changed and the release must not ship.

## 4. Security checks

```
npm audit --production
sha256sum dist/1.0.0/peralys.MassImporterReview.mpk
```

Nothing at CVSS 7.0 or above may ship. This check is required for every version, not only the first.

Upload the `.mpk` to VirusTotal and keep the scan result before submitting to the Marketplace.

## 5. Publish

1. Cut a GitHub release in this repository, tagged with the widget version.
2. Attach the `.mpk` as a **release asset**. Never commit a `.mpk` into the tree; `.gitignore` blocks this deliberately, because a committed artifact drifts out of sync with the build that produced it.
3. Put the SHA-256 of each attached `.mpk` in the release notes.
4. Update `docs/MassImporterReview.md` or `docs/MassImporterUpload.md` if any widget property changed. The widget XML is the only source of truth for those tables.

## 6. Marketplace

Component Source on the Mendix Marketplace submission is **MPK upload**, so the artifact is uploaded to the Marketplace by hand. It is not synced from this repository's releases.
