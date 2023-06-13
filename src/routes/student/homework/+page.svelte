<script>
	import { onMount } from 'svelte';
	import { getFirestore, collection, onSnapshot, query, orderBy, where } from 'firebase/firestore';
	import { initializeApp } from 'firebase/app';
	import { firebaseConfig } from '$lib/config.js';
	import { goto } from '$app/navigation';

	let db = null;
	let homeworkList = [];

	onMount(() => {
		// Initialize the Firebase app
		const app = initializeApp(firebaseConfig);
		db = getFirestore(app);

		// Retrieve the homework list from Firestore
		const homeworkQuery = query(
			collection(db, 'homework'),
			where('dueDate', '>=', new Date().toISOString()),
			orderBy('dueDate')
		);
		const unsubscribe = onSnapshot(homeworkQuery, (snapshot) => {
			homeworkList = snapshot.docs.map((doc) => ({
				id: doc.id,
				...doc.data()
			}));
		});

		// Clean up the listener on component unmount
		return () => {
			unsubscribe();
		};
	});

	function goToSubmissionPage(homeworkId) {
		goto(`/student/homework/submit/${homeworkId}`);
	}
</script>

<main>
	<h2>Homework List</h2>
	{#each homeworkList as homework}
		<div class="homework-item" key={homework.id}>
			<h3>{homework.homeworkName}</h3>
			<p>Task: {homework.task}</p>
			<p>Due Date: {homework.dueDate}</p>
			<p>Completed By: {homework.completedBy.join(', ')}</p>
			<p>For {homework.class} class</p>
			<button on:click={() => goToSubmissionPage(homework.id)}>Submit Homework</button>
		</div>
	{/each}
</main>
