Setup:

put this in a `.env` file in root directory

```
PUBLIC_PROD=false
PUBLIC_CREATOR_HAS_WIFI=true
```

set PUBLIC_CREATOR_HAS_WIFI to enable offline development

Install all deps (pnpm is preferred, but you can use npm or yarn)

```
npm install
```

Run in dev mode (run `npm run expose` to expose on local network, this must be enabled if you are using vscode liveshare)

```
npm run dev
```