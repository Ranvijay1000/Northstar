# Deploy Northstar to Netlify

The frontend is plain HTML, CSS and vanilla JavaScript. No framework or npm dependencies are required. All project files stay in this folder.

## Deploy from Git (recommended)

1. Put this folder's contents in your Git repository. Do not commit `.env` or credentials.
2. In Netlify, choose **Add new project / Import an existing project** and select that repository.
3. If this folder is nested in the repository, set the base directory to its repository-relative path. Otherwise leave it empty.
4. Netlify reads `netlify.toml`: build command `npm run build`, publish directory `public`, functions directory `netlify/functions`, Node.js 22.
5. Deploy. Open the HTTPS site URL. Study tools work immediately without any environment variables.

Alternatively, from this folder with a Netlify account, run `npx netlify-cli deploy --build --prod` and follow its login/site prompts. This command publishes the app. Uploading only `public` through drag-and-drop provides the study tools but does not deploy the optional AI function.

## Enable Profile > Optional AI

1. Generate a random server session secret locally:

   node -e "console.log(require('node:crypto').randomBytes(32).toString('hex'))"

2. Add the resulting 64-character value as `NETLIFY_AI_SESSION_SECRET` in Netlify environment variables. Include the **Functions** scope (or all scopes). Keep this value private; do not add it to HTML, Git or netlify.toml.
3. Redeploy after setting the variable.
4. In the deployed app, open **Profile > Optional AI**, enter your OpenAI API key and a model you can access, then choose **Save & connect**.

Each browser uses its own provider key. The function verifies model access and encrypts the key into an HttpOnly, Secure, SameSite cookie lasting up to seven days. Page scripts cannot read this cookie. Keys are excluded from study backups and localStorage. Disconnect clears the browser connection. Rotating the session secret invalidates all existing connections. Netlify never uses a shared owner OPENAI_API_KEY in this implementation.

AI requests run through Netlify Functions. Core study records remain in your browser; only data for an explicitly requested AI action goes to the provider. API usage may incur provider charges. Request quotas and caches are per active function instance and reset on cold starts; they are convenience limits, not durable billing controls.

## Move existing study records

On the old local app, choose **Export data**. Open the new Netlify site and choose **Import data**. Review the confirmation before replacing records. Browser storage is separate for each URL, browser and device; deployment does not automatically move or synchronize records. Continue exporting backups regularly.

## Validate before publishing

    npm run check
    npm run build
    npm test

The build validates public assets and prevents obvious private files or provider credentials from entering the publish directory. Automated tests exercise the Netlify request handler with mocked provider responses. Live Netlify deployment and paid AI generation require your own account and credentials.

Official references: https://docs.netlify.com/build/functions/get-started/ and https://docs.netlify.com/build/functions/environment-variables/

## Firebase accounts

The login screen supports sign-in, account creation, password reset and guest access. Follow [FIREBASE.md](FIREBASE.md) to set up your web configuration, Email/Password provider and private Firestore rules before redeploying. Firebase configuration is independent of the optional AI session secret.
