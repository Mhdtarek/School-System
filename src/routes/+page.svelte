<script>
	import { onMount } from 'svelte';
	import { goto } from '$app/navigation';
	import { userStore } from '$lib/stores';

	/**
	 * @type {{ verified?: any; displayName?: any; email?: any; photoURL?: any; }}
	 */
	let user;

	onMount(() => {
		// Subscribe to the userStore to get the user data
		user = $userStore;

		// Check if user data is undefined or empty
		if (!user || Object.keys(user).length === 0) {
			goto('/auth'); // Redirect to auth route
		}
	});

	/**
	 * Check if the user is verified.
	 * @returns {boolean} True if the user is verified, false otherwise.
	 */
	function isUserVerified() {
		return user && user.verified;
	}
</script>

<main class="container">
	{#if user}
		{#if isUserVerified()}
			<div>
				<p>Welcome, {user.displayName}!</p>
				<p>Email: {user.email}</p>
				{#if user.photoURL}
					<img src={user.photoURL} alt="User Profile" />
				{/if}
			</div>
		{:else}
			<p>You are not verified. Please contact the admin for verification.</p>
		{/if}
	{:else}
		<p>No user data available.</p>
	{/if}
</main>
