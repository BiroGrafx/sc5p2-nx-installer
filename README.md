<div align=center>

<img src="icons/icon-192.png" alt="Space Channel 5 Part 2" width="20%">

</div>
<h1 align=center>sc5p2-nx installer</h1>

The installer page of [sc5p2-nx](https://github.com/BiroGrafx/sc5p2-nx), the native Nintendo Switch port of
Space Channel 5 Part 2: **https://BiroGrafx.github.io/sc5p2-nx-installer/**

You give it your own copy of the Xbox Live Arcade game (its content package, or a folder you extracted it into) and it
gives you a zip ready to extract into `sdmc:/switch/`. Everything runs in your browser: the game files never leave
your computer.

It is [sa2-nx-installer](https://github.com/ChanseyIsTheBest/sa2-nx-installer) by ChanseyIsTheBest, set up for this
game: the same code, its WebAssembly tools unchanged, and this game's package, shaders and build list.

## What it does

1. Finds the game in what you chose: the content package `B1A42250167C9606EE7B7C6491AFD506B839176858` (read in
   place, nothing is extracted to disk), the `58410A39` folder of an Xbox 360's `Content` folder, or an extracted game
   with its `default.xex`.
2. Identifies the game by the SHA-256 of `default.xex` (or of its decompressed image, for an executable another tool
   rebuilt) and picks its build from `release/manifest.json`.
3. Downloads that build's NRO and checks its SHA-256.
4. Makes the shader library from the game's own files: the shader containers of `ch52_g/ShaderXbox360`, and
   Direct3D's built-in shaders, whose microcode is read from the decompressed `default.xex` and put into containers
   exactly as the tested library has them (templates `BillBoard.wvu` and `SimpleMultiModel.wpu`). It translates the
   Xbox 360 microcode to HLSL (XenosRecomp), compiles it to SPIR-V (DXC) and packs the library. The library must match,
   byte for byte, the one the NRO was tested with.
5. Writes the game files as the asset pack the console mounts (`game_root.nxpack` + `game_root.nxidx`), the format of
   the SDK's PackDevice. The console doesn't have to build it on its first start.
6. Streams the zip to disk through the service worker, so the 1.8 GB package never has to fit in memory.

The zip holds `sc5p2-nx/sc5p2.nro` with its `LICENSES.txt`, `sa2_shaders.mesp` + `.idx` and `game_root.nxpack` +
`.nxidx`. The update zip holds the NRO, its licenses and the shaders. Neither holds `sc5p2.toml`: the game writes it
at its first start, so an update never overwrites your settings.

The page and the downloaded program are cached by the service worker: after the first visit it also works without a
connection.

## Layout

| Path | Contents |
|---|---|
| `index.html`, `app.js` | The page and its interface (six languages, `lib/i18n.js`) |
| `worker.js` | Builds the package off the main thread |
| `sw.js` | Offline cache and the download stream |
| `lib/` | STFS package reader, XEX reader, shader containers and library, asset pack writer, zip writer |
| `wasm/` | WebAssembly builds of the shader translator, DXC, the library packer and the LZX decoder |
| `shader_common.h` | The translator's HLSL header (from `shaders/XenosRecomp` of sa2-nx) |
| `release/` | The list of builds and the licenses of the NRO |
| `test/` | Node scripts and a browser page that run the same code as the page |

`wasm/` is sa2-nx-installer's, unchanged: `wasm/hlsl.*`, `wasm/pack.*` and `wasm/lzx.*` are built from the sa2-nx
repository with `shaders/wasm/build_wasm_tools.sh` (Emscripten); `wasm/dxc_web.*` is the WebAssembly DXC of
[nfsmw-nx-installer](https://github.com/StevensND/nfsmw-nx-installer). See [NOTICES.md](NOTICES.md).

## Deployment

The NRO is not stored here: it is the `sc5p2.nro` asset of a release of the sc5p2-nx repository of the same owner.
The workflow in `.github/workflows/deploy.yml` downloads it from the latest release, checks its SHA-256 against the
list and publishes the page on GitHub Pages. It runs on every push to `main`; after publishing a new release, run it
by hand from the Actions tab. Pages must be set to deploy from GitHub Actions (Settings > Pages > Source).

To publish a new build, update the build's entry in `release/manifest.json`. This script fills in the NRO's SHA-256
and size and the fingerprints of the game and of the shader library, made from your own copy with the page's code:

```sh
node test/release_node.mjs <game package> <sc5p2.nro> --version 1.1.0 --write
```

Test the NRO with exactly that library first: it is `sa2_shaders.mesp` of a zip made by `test/package_node.mjs`.

## Testing locally

Put `sc5p2.nro` in `release/` and serve the folder (any static server works; this one sets the right MIME types):

```sh
python serve.py 8193
```

Then open http://127.0.0.1:8193/. The same code runs in Node:

```sh
node test/package_node.mjs sc5p2-nx.zip <game package or folder>
node test/package_node.mjs sc5p2-nx-update.zip <game package or folder> --update
node test/library_node.mjs <game package or folder> [<folder of reference containers>]
node test/texts_node.mjs
node test/fnv_node.mjs
```

## License

GPL-3.0 (see [LICENSE](LICENSE)). Third-party components are listed in [NOTICES.md](NOTICES.md). The page is
[sa2-nx-installer](https://github.com/ChanseyIsTheBest/sa2-nx-installer) by ChanseyIsTheBest, itself built on
[nfsmw-nx-installer](https://github.com/StevensND/nfsmw-nx-installer) by StevensND.

Space Channel 5 and Ulala are trademarks of SEGA; the icon artwork is © SEGA. This project is not affiliated with or
endorsed by SEGA, Nintendo or Microsoft.
