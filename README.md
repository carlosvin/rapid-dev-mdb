# Fullstack App focused on rapid development

When you create a Prove of Concept, a pilot, a prototype, normally you want to have a piece of software tha just works, the main goal is not having a production read app at all, it is to demonstrate the feasibility and viability of an idea, technology, or product before investing significant resources into its development.

In this repository I am exemplifying how easy is to do rapid development with [SvelteKit](https://kit.svelte.dev/) (fullstack framework) and [MongoDb](https://www.mongodb.com) as database.

## Creating a Sveltekit project

```bash
# create a new project in the current directory
npm create svelte@latest

# create a new project in my-app
npm create svelte@latest my-app
```

### Developing

Once you've created a project and installed dependencies with `npm install` (or `pnpm install` or `yarn`), start a development server:

```bash
npm run dev

# or start the server and open the app in a new browser tab
npm run dev -- --open
```

### Building

To create a production version of your app:

```bash
npm run build
```

You can preview the production build with `npm run preview`.

> To deploy your app, you may need to install an [adapter](https://kit.svelte.dev/docs/adapters) for your target environment.

## Integrating SvelteKit with MongoDB

The MongoDB integration should happen on the server side only.

To make sure we have a connection to the DB active we create a [`src/hooks.server.ts`](./src/hooks.server.ts) file.

I have created a simple abstraction [`./src/lib/services/db.ts`](./src/lib/services/db.ts) to easily connect and access to the collections once we have established a connection to the database.

### Read connection credentials from environment

#### Local development

You just need to make sure that the MongoDB URL and the MongoDB database name are configured in the .env.local file, make sure that you add this file to the .gitignore so that you don't push it by mistake and share your secrets.

```conf
VITE_DB_CONN='mongodb+srv://<your user name>:<password>@myfree.domain.mongodb.net'
VITE_DB_NAME='rotationDb'
```

Note that we are reading this variables in the [`src/hooks.server.ts`](./src/hooks.server.ts) file.

#### App Deployment

When you deploy your application, you just need to make sure that the secrets are passed to the deployed server as environment variables.

It really depends on the provider, I highly recommend using one of the [officially supported by svelte as adapters](https://kit.svelte.dev/docs/adapters), just for the shake of simplicity.

## Connecting to MongoDB (Free)

You need the MongoDB connection URL and the database name.

You can just have MongoDB installed locally or even simpler, just get a [Free Tier cluster](https://www.mongodb.com/docs/atlas/getting-started/). Remember that in one cluster you can have several databases in case you want to use the same cluster for different PoCs/prototypes.

You just need to register (easy with your github or google accounts) and create a free cluster.
![My Free cluster](./imgs/atlas-free.png)

Once you have the free cluster, you can create different DBs, in this screenshot I have the poc1 db and the rotationDb.
![Different DBs](./imgs/atlas-collections.png)

### Database access

You can just configure the DB access in the same section.
![DB access](./imgs/db-access.png)

### Database connect

Now it is super easy to connect, just click on the database menu entry on the left, click connect and follow the instructions.
![DB connect](./imgs/db-connect.png)

## Fullstack development

Sveltekit solution is quite straightforward:

- [`+page.svelte`](./src/routes/teams/+page.svelte) files for client or frontend code.
- [`+page.server.ts`](./src/routes/teams/+page.server.ts) files for the code meant to run on the server only.

```typescript
// this is running on the server `+page.server.ts`
export const load: PageServerLoad = async () => {
	// it reads the items from the database
	const documents = await DB.teams.find<TeamDb>({}).toArray();

	// it converts the documents to app objects
	const teams = documents.map<Team>(({ name, _id }) => ({
		name,
		id: _id.toString()
	}));

	// this can be used later on the browser (+page.svelte)
	return { teams };
};
```

```svelte
<script lang="ts">
 // running on the browser
 // page.svelte file
 import type { PageData } from './$types';

 export let data: PageData;
</script>

<h1>Teams list</h1>
{#if data.teams.length === 0}
    <p>No teams</>
{/if}
```

This documentation is using simplified versions of the code, but you can check and run the whole example in this repository.
