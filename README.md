# Clerk SvelteKit

`clerk-sveltekit` is a community maintained [Clerk](https://clerk.com/) authentication integration for [SvelteKit](https://kit.svelte.dev/). Visit a live [demo](https://clerk-sveltekit.markjaquith.com/), which is just this repository hosted on Cloudflare Pages.

## Installation

Add `clerk-sveltekit` with your package manager of choice:

```shell
npm install clerk-sveltekit
```

```shell
pnpm add clerk-sveltekit
```

```shell
yarn add clerk-sveltekit
```

```shell
bun add clerk-sveltekit
```

## Usage

### Set up environment variables

If you haven't already, create a `.env` file at the root of your application. Inside, add these two environment variables:

```env
PUBLIC_CLERK_PUBLISHABLE_KEY=pk_test_xxx
CLERK_SECRET_KEY=sk_test_xxx
```

These keys can always be retrieved from the [API keys](https://dashboard.clerk.com/last-active?path=api-keys) page of your Clerk Dashboard. Inside the dropdown choose "Astro" and copy/paste the values.

**Note:** For production instances using OAuth providers, you will have to do some more setup with Clerk and DNS.

### Configure the server hook

Add this to `src/hooks.server.ts` (or integrate this code with your existing `hooks.server.ts` file):

```ts
import { withClerkHandler } from 'clerk-sveltekit/server'

export const handle = withClerkHandler()
```

Update the `app.d.ts` file to ensure that the locals added by the Clerk handler are properly typed.

```ts
/// <reference types="clerk-sveltekit/env" />

declare global {
	namespace App {...}
}
```

### Configure the client hook

Add this to `src/hooks.client.ts`:

```typescript
import { initializeClerkClient } from 'clerk-sveltekit/client'
import { PUBLIC_CLERK_PUBLISHABLE_KEY } from '$env/static/public'

initializeClerkClient({
	publishableKey: PUBLIC_CLERK_PUBLISHABLE_KEY,
	signInForceRedirectUrl: '/admin',
	signUpForceRedirectUrl: '/admin',
	signInUrl: '/sign-in',
	signUpUrl: '/sign-up',
})
```

Customize the protected paths, and the various URLs as you like.

### Use the components

Next, put the `<SignIn />` component on your sign in page:

```svelte
<script lang="ts">
	import SignIn from 'clerk-sveltekit/client/SignIn.svelte'
</script>

<div>
	<SignIn />
</div>
```

And place the `<SignUp />` component on your sign up page:

```svelte
<script lang="ts">
	import SignUp from 'clerk-sveltekit/client/SignUp.svelte'
</script>

<div>
	<SignUp />
</div>
```

Then, where you want to show the signed-in user's photo and sign out button (probably in a `+layout.svelte` file in the header):

```svelte
<script lang="ts">
	import UserButton from 'clerk-sveltekit/client/UserButton.svelte'
	import SignedIn from 'clerk-sveltekit/client/SignedIn.svelte'
	import SignedOut from 'clerk-sveltekit/client/SignedOut.svelte'
</script>

<SignedIn>
	<UserButton afterSignOutUrl="/" />
</SignedIn>
<SignedOut>
	<a href="/sign-in">Sign in</a> <span>|</span> <a href="/sign-up">Sign up</a>
	<!-- You could also use <SignInButton mode="modal" /> and <SignUpButton mode="modal" /> here -->
</SignedOut>
```

## Components

All components can be imported from `clerk-sveltekit/client/ComponentName.svelte`

- `<ClerkLoading />` — Wrapper that shows its contents when Clerk is still loading.
- `<ClerkLoaded let:clerk />` — Wrapper that shows its contents (and exposes the `clerk` object) when Clerk is done loading.
- `<SignIn />` — Renders a sign-in form.
- `<SignUp />` — Renders a sign-up form.
- `<SignedIn let:user />` — Wrapper that shows its contents (and exposes the Clerk `user` object) when the user is signed in.
- `<SignedOut />` — Wrapper that shows its contents when the user is not signed in.
- `<UserButton />` — Button that shows the user’s profile photo with log out link when they are signed in.
- `<UserProfile />` — Renders the current user’s profile.
- `<SignInButton />` — Unstyled sign-in button (can do `mode="modal"` too).
- `<SignUpButton />` — Unstyled sign-up button (can do `mode="modal"` too).
- `<SignOutButton />` — Unstyled sign-out button.
- `<OrganizationProfile />` — Renders the organization profile component.
- `<OrganizationSwitcher />` — Renders an organization switcher component.
- `<CreateOrganization />` — Renders UI for creating an organization.
- `<Protect />` — Wrapper that shows its contents when the current user has the specified [permission or role](https://clerk.com/docs/organizations/roles-permissions) in the organization.

**Note:** Components should be used for displaying UI, but are not sufficient for protecting routes. To protect a route, you check the value of `userId` in the `auth` local and redirect if it is not set, for example.

```ts
import { redirect } from '@sveltejs/kit'
import { clerkClient } from 'svelte-clerk/server'

export const load = ({ locals }) => {
	if (!locals.auth.userId) {
		return redirect(307, '/sign-in')
	}

	const user = await clerkClient.users.getUser(userId)

	return {
		user: JSON.parse(JSON.stringify(user)),
	}
}
```

## Using Clerk data on the server

Server-side protected routes will automatically get a [Clerk user object](https://clerk.com/docs/references/javascript/user/user) injected into `locals.auth` which means you can use it [in a `load()` function](https://kit.svelte.dev/docs/form-actions#loading-data), a [default action](https://kit.svelte.dev/docs/form-actions#default-actions), or a [form action](https://kit.svelte.dev/docs/form-actions).

## Thanks

Thanks to Cerbos for their [https://github.com/cerbos/sveltekit-clerk-cerbos](sveltekit-clerk-cerbos) example repo which got this project started, and to [Brian Bug](https://thebrianbug.com/) for fixing bugs in that implementation.
