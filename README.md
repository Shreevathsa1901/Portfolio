# Shreevathsa Bhat — Portfolio

A personal portfolio for Shreevathsa Bhat, an MCA student at PES University working toward AI and machine-learning engineering.

## Run locally

```bash
npm install
npm run dev
```

Open [http://localhost:3000](http://localhost:3000).

## Validate

```bash
npm run lint
npm run build
```

## Deploy to Vercel

This is a Next.js application and can be deployed from a Git repository using Vercel. This workspace is not currently connected to a Git repository:

1. Create a GitHub repository you control and sign in to GitHub in your terminal.
2. From this folder, initialize and push the project:

   ```bash
   git init -b main
   git add .
   git commit -m "Prepare AI and ML portfolio"
   git remote add origin https://github.com/<your-username>/<your-repository>.git
   git push -u origin main
   ```

3. Import the repository at [vercel.com/new](https://vercel.com/new), keep the detected Next.js settings, and deploy.
4. To enable the portfolio chat, add `OPENROUTER_API_KEY` in the Vercel project's Environment Variables, then redeploy. The site can build without this key, but chat requests will remain unavailable until it is configured.

The `.gitignore` excludes dependencies, build output, `.env` files, and the local LinkedIn PDF. Never commit API keys or paste them into chat; configure secrets only in the hosting provider's environment settings.
