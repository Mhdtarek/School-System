<script>
	import { onMount } from 'svelte';
	import { getFirestore, doc, onSnapshot } from 'firebase/firestore';
	import { initializeApp } from 'firebase/app';
	import { firebaseConfig } from '$lib/config.js';

	let db = null;
	export /**
	 * @type {{ link: any; homeworkName?: any; task?: any; dueDate?: any; completedBy?: any; }}
	 */
	let data;

	let homeworkNotFound = false;

	onMount(() => {
		// Initialize the Firebase app
		const app = initializeApp(firebaseConfig);
		db = getFirestore(app);

		// Retrieve the homework item from Firestore based on the slug
		const homeworkDocRef = doc(db, 'homework', data.link);
		const unsubscribe = onSnapshot(homeworkDocRef, (doc) => {
			if (doc.exists()) {
				// Update the data object with the homework details
				data = { ...doc.data(), link: data.link };
			} else {
				// Handle case where the homework item doesn't exist
				homeworkNotFound = true;
			}
		});

		// Clean up the listener on component unmount
		return () => {
			unsubscribe();
		};
	});
</script>

<main>
	{#if !homeworkNotFound}
		<h2>Homework Details</h2>
		{#if data}
			<div>
				<h3>{data.homeworkName}</h3>
				<p>Task: {data.task}</p>
				<p>Due Date: {data.dueDate}</p>
				{#if data.completedBy}
					<p>Completed By: {data.completedBy.join(', ')}</p>
				{/if}
			</div>
		{:else}
			<p>Loading homework details...</p>
		{/if}
	{:else}
		<p>Homework not found.</p>
	{/if}
</main>
