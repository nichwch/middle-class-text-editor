## Middle Class Text Editor

Slack style mentions implemented in a plain text area.



https://github.com/nichwch/middle-class-text-editor/assets/7423703/9b524177-19c5-4a26-9795-96b670cd2ef7

## How does it work?
Great question! Short answer: By abusing absolute positioning.

Long answer: I use three elements positioned absolutely so that they're all overlapping: an "underlay div", the textarea, and an "overlay div".

The textarea is where the user types. As they type, the underlay clones the content one for one, but renders keywords differently using a regex match.

The overlay renders all the keywords (again, using a regex match), and positions them absolutely to match their corresponding keyword in the underlay. I use a separate overlay instead of rendering keywords directly in the underlay because otherwise the z-index and stacking context would prevent the keywords from being clickable.

Here's a handy visualization of the setup.

![explanation](https://github.com/nichwch/middle-class-text-editor/assets/7423703/6ab9cc70-6b32-499d-bbcc-c93659b5fe7e)


## Developing

Once you've created a project and installed dependencies with `npm install` (or `pnpm install` or `yarn`), start a development server:

```bash
npm run dev

# or start the server and open the app in a new browser tab
npm run dev -- --open
```

Everything inside `src/lib` is part of your library, everything inside `src/routes` can be used as a showcase or preview app.

## Building

To build your library:

```bash
npm run package
```

To create a production version of your showcase app:

```bash
npm run build
```

You can preview the production build with `npm run preview`.

> To deploy your app, you may need to install an [adapter](https://kit.svelte.dev/docs/adapters) for your target environment.

## Publishing

Go into the `package.json` and give your package the desired name through the `"name"` option. Also consider adding a `"license"` field and point it to a `LICENSE` file which you can create from a template (one popular option is the [MIT license](https://opensource.org/license/mit/)).

To publish your library to [npm](https://www.npmjs.com):

```bash
npm publish
```
