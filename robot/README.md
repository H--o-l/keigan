# create-svelte

Everything you need to build a Svelte project, powered by [`create-svelte`](https://github.com/sveltejs/kit/tree/master/packages/create-svelte).

## Creating a project

If you're seeing this, you've probably already done this step. Congrats!

```bash
# create a new project in the current directory
npm init svelte

# create a new project in my-app
npm init svelte my-app
```

## Developing

Once you've created a project and installed dependencies with `npm install` (or `pnpm install` or `yarn`), start a development server:

```bash
npm run dev

# or start the server and open the app in a new browser tab
npm run dev -- --open

# or to serve on local network
npm run dev -- --host
```

## Building

To create a production version of your app:

```bash
npm run build
```

You can preview the production build with `npm run preview`.

> To deploy your app, you may need to install an [adapter](https://kit.svelte.dev/docs/adapters) for your target environment.


# For the accelerometer during dev

1. On your Android device, open the Chrome app.
2. In the address bar, type chrome://flags and press Enter.
3. In the search bar, type unsafely-treat-insecure-origin-as-secure and press Enter.
4. From the dropdown menu, select the Enabled option.
5. In the text field next to the dropdown menu, type the IP address of the server that you want to use the Accelerometer interface on, for example http://192.168.0.100:8000.
6. Click the Relaunch button to restart Chrome with the flag enabled.

